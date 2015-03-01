---
layout: post
title: Side Project - Flatiron Twitter CLI
---

Last week, Avi pitched a pretty cool idea to the class: build an "auto-follow bot" for Flatiron student twitter accounts - something we could use to easily auto-follow everyone in the class.

![Avi Flatiron Twitter pitch]({{ site.baseurl }}/assets/avi-pitch.png "Avi Flatiron Twitter pitch")

I'd been looking for a side project to work on (inspired by my classmates who've built some very sweet side-projects already: [ex.1](http://rebecca-eakins.github.io/2015/02/23/how-to-combat-writers-block-and-boost-noob-cred.html), [ex.2](http://www.thegreatcodeadventure.com/weather-for-dummies/), [ex.3](http://www.thegreatcodeadventure.com/sinatra-gets-me-all-the-cats/), [ex.4](http://www.seijinaganuma.com/2015/02/scrape-it-ball/)) and this seemed like a perfect opportunity. I've never built a bot, but I know how to build a CLI, so I put labwork on pause and spent my Saturday building the [FLATIRON TWITTER CLI](https://github.com/ktravers/flatiron-twitter-cli) instead (available at your [local github](https://github.com/ktravers/flatiron-twitter-cli)).

Let's walk through the build, starting with setup. I tried to follow best practices, setting up bin, config, and lib folders, along with a Gemfile and README. 

Right off the bat, my `bin/run` file almost sunk me. I wanted users to be able to initialize the CLI by simply cd'ing into the directory and running `bin/run`.

```ruby
#!/usr/bin/env ruby
require_relative '../config/environment'

FlatironTwitterCLI.new.call
```

I thought the code above would do it, but every time I ran it, I got a nasty -bash error:  
`-bash: bin/run: Permission denied`

Google searching told me this error pops up when there's something off with your $PATH, but I couldn't really decipher any of the solutions that were offered on the various forums. Luckily, I remembered a few of my classmates had run into similar problems with their CLIs, so I checked their repos for clues. Jeremy and Alex's [Stockvestigator3000](https://github.com/jeremysklarsky/stock-cli) had the solution: I need to run `chmod +x bin/run`. This nice little command "change[s] the permission of the file ... to be executable", according to [this source](http://www.cyberciti.biz/faq/howto-unix-command-run-execute-bin-files-in-linux/). Problem solved.

Inside `config`, my environment file is pretty straight-forward:

```ruby
require 'Bundler'
Bundler.require

require 'nokogiri'
require 'open-uri'
require 'json'

require_all 'lib'
```

Likewise with my `Gemfile`:

```ruby
source "https://rubygems.org"

gem 'pry'
gem 't'
gem 'nokogiri'
gem 'require_all'
```

The showpiece here is the `t gem`. Originally included as part of the [twitter gem](http://sferik.github.io/twitter/), this command-line-specific interface was removed early on and rebuilt by [sferik](https://github.com/sferik) as a separate gem. It offers full Twitter-functionality from the command line, including the ability to "mass follow" people using the `t follow(*users)` command. My goal here was simply to extend that funcationality a tiny bit by adding a couple Flatiron-specific commands.

Moving on to `lib`, which contains my controllers and models. 

My classmates will recognize the bulk of the code in the `StudentScraper` and `FlatironTwitterCLI` classes. `StudentScraper` should be nearly identical to the one we built for our scraping-the-students-page lab, and `FlatironTwitterCLI` uses most of the methods developed in the various Playlister labs, with a couple modifications.

I chose to load `@students` and `@instructors` at initialization. This adds some lag to the load time, so I need to look into better ways to optimize that.

```ruby
class FlatironTwitterCLI
  attr_accessor :on, :students, :instructors #, :staff
  APPROVED_COMMANDS = [:help, :list, :follow, :exit]

  def initialize
    @students = StudentScraper.new.call        ## load students at init
    @instructors = InstructorScraper.new.call  ## load instructors at init
    # staff = StaffScraper.new.call            ## more on StaffScraper later...
    @on = true
    system("clear")
    Welcome.new.call
  end
```

The CLI responds to its own built-in commands ('help', 'list', 'follow', 'exit'), as well as any of the `t gem` commands. I did this by adding a simple check into the `command(input)` method.

```ruby
  def get_command
    puts "\nPlease enter a valid command."
    puts "(Type 'help' to see a list of valid commands)"
    self.command_request
  end

  def command(input)              ## allows users to access t gem commands
    if input.slice(0,2) == "t "   ## otherwise runs input through command_valid? method
      system(input)
    else
      send(input) if command_valid?(input) 
    end
  end

  def command_valid?(input); APPROVED_COMMANDS.include?(input.downcase.to_sym); end
  def command_request; self.command(user_input); end
  def user_input; gets.downcase.strip; end 
```

Now onto the exiting stuff: scraper classes. Initially, I thought it'd be a breeze to set up separate scrapes for students, instructors and staff (using the [student index page](http://ruby007.students.flatironschool.com/) and [Flatiron Staff page](http://flatironschool.com/team#staff)). Sadly, my CSS / Nokogiri skills aren't where I thought they were, because I was only able to successfully scrape students and instructors. You'll see in my [StaffScraper file](https://github.com/ktravers/flatiron-twitter-cli/blob/master/lib/models/staff_scraper.rb), I'm trying to iterate through each of the `div.person-box`es in `section#staff`, but for some reason, when I tap-create Staff objects, it creates an instance for each staff member AND each instructor. I spent the better part of my Saturday night fighting with it, but I couldn't figure out how to _not_ select the instructors. Please - better Ruby coders - help me fix this!

```ruby
class StaffScraper

  def call
    html = open("http://flatironschool.com/team")
    staff_doc = Nokogiri::HTML(html)

    ## WHY DO INSTRUCTORS GET INCLUDED IN THIS SEARCH? 
    staff_doc.search('#staff div:nth-child(2) div.person-box').collect do |member|
      name = member.search('h2').text
      role = member.search('strong.title').text

      social = member.search('ul.social-networks li a').collect { |link| link['href'] }
      twitter = social.select { |link| link.include?("twitter") == true}.first

      if twitter != nil
        twitter = twitter.gsub(/(http:\/\/twitter.com\/|https:\/\/twitter.com\/)/,"@")
      else
        twitter = "@flatironschool"
      end

      Staff.new.tap do |staff|
        staff.name = name
        staff.twitter = twitter
        staff.role = role
      end
    end
    #=> CREATES 39 STAFF INSTANCES: 24 STAFF & 15 INSTRUCTORS
  end
end
```

![April Ludgate Cry for Help]({{ site.baseurl }}/assets/aprilhelp.gif "Cry for Help")

Once we get the the StaffScraper fixed, we'll be able to uncomment the `staff` methods/commands inside `FlatironTwitterCLI` to add the capability to list & follow staff members. Until then, just students and instructors are working.

One other error I'd like to tamp down is this open-uri RuntimeError that happens every now and again after running `bin/run`:

```
/Users/ktmoney/.rvm/rubies/ruby-2.1.5/lib/ruby/2.1.0/open-uri.rb:231:in `open_loop': HTTP redirection loop: http://flatironschool.com/team (RuntimeError)
  from /Users/ktmoney/.rvm/rubies/ruby-2.1.5/lib/ruby/2.1.0/open-uri.rb:149:in `open_uri'
  from /Users/ktmoney/.rvm/rubies/ruby-2.1.5/lib/ruby/2.1.0/open-uri.rb:704:in `open'
  from /Users/ktmoney/.rvm/rubies/ruby-2.1.5/lib/ruby/2.1.0/open-uri.rb:34:in `open'
  from /Users/ktmoney/Documents/Documents/Flatiron/flatiron_projects/flatiron-twitter/lib/instructor_scraper.rb:7:in `call'
  from /Users/ktmoney/Documents/Documents/Flatiron/flatiron_projects/flatiron-twitter/lib/controllers/flatiron_twitter_cli.rb:8:in `initialize'
  from bin/run:5:in `new'
  from bin/run:5:in `<main>'
  ```

If anyone can shed light on that issue, it'd be much appreciated.

Finally, this thing needs lots of refactoring overall. My [README](https://github.com/ktravers/flatiron-twitter-cli#todo) lists a couple fixes/feature builds I'd like to to get in place. Contributions welcome - let's get this thing polished up! Then once it's bright and shiny, I'll figure out how to turn it into a gem (but that's a topic for another post...)

Thanks for checking this out, and hope you enjoy using the [Flatiron Twitter CLI](https://github.com/ktravers/flatiron-twitter-cli)!
