---
layout: post
title: Who's Ahead Now?
---

After week one of learning Rails, I was eager to test out my new skills on a new side project. I wanted to build something fairly simple, that would utilize skills I felt comfortable with - JSON parsing, ERB, simple search and routing - while also pushing me into uncharted territory, like deploying on [Heroku](https://www.heroku.com). I spent the afternoon brainstorming, circling around some sort of app that would answer a single simple question (like [Is It Raining?](http://isitraining.in/New-York), one of my fave [single serving sites](http://kottke.org/08/02/single-serving-sites)). I had a couple good ideas, but my partner came up with the best one: is Hillary winning? 

Now obviously a website that just says "aw yass" every time it renders doesn't have much utility. So I decided to expand the concept a bit and build a site that yields the current frontrunners for each party in the current presidential race. A quick [rubygems](https://rubygems.org) search pointed me to [HuffPost's Pollster API](http://elections.huffingtonpost.com/pollster/api), which is really nicely maintained. Armed with data and a plan, I started building.

[![Texts from Hillary]({{ site.baseurl }}/assets/hillpost.jpg "HillaryPost")](http://textsfromhillaryclinton.tumblr.com/post/20838007265/original-image-by-diana-walker-for-time "HillaryPost")

The app ended up being pretty simple:  
- **One model** (Search)  
- **Three controllers** (ApplicationController, Search_Controller, and WelcomeController, which has zero methods)  
- **Two views** (plus header/footer layouts)  
- **Five routes** (some of which I can probably get rid of, actually):   

```bash
$ rake routes
      Prefix Verb   URI Pattern                Controller#Action
        root GET    /                          welcome#index
search_index GET    /search(.:format)          search#index
             POST   /search(.:format)          search#create
  new_search GET    /search/new(.:format)      search#new
      search GET    /search/:id(.:format)      search#show
```

You'll notice there's no database, so no need to use ActiveRecord (or anything beyond routing and rendering). I'm not storing the poll data; just sending a request to Huffpost's API and rendering that data in a separate view. Originally, I'd mapped out ActiveRecord relations where parties have many candidates, and candidates have many poll results, etc., etc., but it quickly became clear none of that was necessary.

The bulk of the logic is in the `Search` model. I applied a lot of the same methods here that we used with the Spotify API, most importantly breaking down the parsing into small, single service methods (like `get_url` and `get_json`). I've listed just the methods I used to get the current leader below. You can visit the [source code](https://github.com/ktravers/whos-ahead-now) to see other methods for `standings` and `race_url`.

```ruby
class Search

  BASE_URL = "http://elections.huffingtonpost.com/pollster/api"
  
  def get_url(party)
    "#{BASE_URL}/charts/2016-national-#{party}-primary"
    #=> http://elections.huffingtonpost.com/pollster/2016-national-democratic-primary.json
    #=> http://elections.huffingtonpost.com/pollster/2016-national-gop-primary.json
  end

  def get_json(url)
    JSON.load(open(url))
  end

  def get_leader(pollster_hash)
    "#{pollster_hash["estimates"].first["choice"]}"
  end
end
```

The `SearchController` is equally simple. It creates four variables to pass the views: one that it builds from params, passed from the form on the `root_path` (and then sanitized via the private `search_params` method), and three that it builds from the `Search` class.

```ruby
class SearchController < ApplicationController

  def index 
    #=> params = {"utf8"=>"✓", "commit"=>"democratic"}
    if params[:commit]
      @party = params[:commit]
      @leader = Search.new.leader(@party)
      @standings = Search.new.standings(@party)
      @url = Search.new.url(@party)
      render 'show'
    end
  end

  private
    def search_params
      params.require(:search).permit(:commit)
    end
end
```

The views all hinge on a single user action: clicking the Democrat or Republican image on the `welcome#index` page. I learned a new trick here; these images are actually FORMS. Oh yeah, you can do that in Rails. You just use `f.submit "value", :type => :image`, then specify the `image_path`. 

```html
<!-- welcome#index -->

<div class="gop">
  <%= form_for(:search, :url => search_index_path, :method => 'GET' ) do |f| %>
  <%= f.submit "gop", :type => :image, :src => image_path("republican.png") %>
  <% end %>
</div>
```

Unlike a standard link, clicking the form/image now sends that `"commit" => "value"` as a param to the target page, where I can use it to do all kinds of fun conditional displays. The `search#show` view is set up so that most details of the page, everything from the actual content to the background color, is all determined by that `commit param`.

```
<!-- search#show -->

<%= <body id="gop"> if @party == "gop" %>
<%= <body id="dem"> if @party == "dem" %>

    <div class="leader"><%= @leader %>!<br></div>

    <div class="standings">
      <table>
        <tr>
          <th>Candidate</th>
          <th>Percentage</th>
        </tr>
        <tr>
        <% @standings.each do |candidate| %>
          <td><%= candidate.first %></td>
          <td><%= candidate.last %>%</td>
        </tr>
        <% end %>
      </table>
    </div>
  
    <%= link_to "Detailed polling results", @url %>
```

The thing that was actually hardest for me to pull off was the CSS hover effects for the two form/images on the landing page. Since they're actually `form type=image` and not just straight-forward images, it was a little more difficult to apply the [usual fun hover effects](http://gudh.github.io/ihover/dist/).* I ended up using a vector image on the form with transparent elements, then using CSS to add a background image on hover that would then fill in those transparent elements in the top image. It's not the most artful solution, but it'll work for now.

```css
/* stylesheets/style.css */

div.dem input[type=image]:hover,
div.dem input[type=image]:active,
div.dem input[type=image]:focus {
  background-image: url('democrat-bg.png');
  background-position: 100% 50%;
  background-repeat: no-repeat;
}
```

_*ed. note:_ In researching links for the paragraph above, I think I just stumbled across a much better solution [here](http://stradegyadvertising.com/tutorial-how-to-image-hover-swap-css/), using "hover swap" and the `background-position` attribute. I'll try to put that in place next week.

Next challenge was deploying to [Heroku](https://www.heroku.com), something I'd never done before. I started with their [Ruby walkthrough](https://devcenter.heroku.com/articles/getting-started-with-ruby#introduction) and was quickly sidelined on the ["Run the app locally" step of the demo](https://devcenter.heroku.com/articles/getting-started-with-ruby#run-the-app-locally). My PostgreSQL configuration was off, then after I `brew uninstall`ed and re-installed, I started getting authentication errors ... things were getting ugly. Briefly discouraged, I thought to myself, "What would Hill-dawg do?" She didn't let The Patriarchy stop her; no way she'd let some Postgres db-nonsense get her down.

![Hillary Deal With It]({{ site.baseurl }}/assets/dealwithit.gif "Hillary Deal With It")

So I went forward and deployed anyway, following the steps below:

1. [Install Heroku](https://devcenter.heroku.com/articles/getting-started-with-ruby#set-up) and login using `heroku login`.  
2. `cd` into your app directory  
3. Run `heroku create [app name]` from your app's root directory  
4. Run `git add .` and `git commit -m "message"` same as usual  
5. Run `git push heroku master` to push to Heroku. If all goes well, the last line should read `http://your-app-name.herokuapp.com/ deployed to Heroku`  
6. Key in `heroku ps:scale web=1` to make sure an instance of the app is running  
7. Run `heroku open` to open up http://your-app-name.herokuapp.com/ in the browser  

The last mile was connecting my app to a custom URL. Setup on the Heroku side of things was pretty simple. Just run `heroku domains:add www.example.com` from the command line to attach the URL to the app. Then go into your DNS settings and add a CNAME record pointing your www subdomain to your Heroku hostname (your-app-name.herokuapp.com). Wait for the servers to propogate, and you're done.

![Happy Hillary](http://38.media.tumblr.com/tumblr_m46rufsN9r1r3whbto1_500.gif "Happy Hillary gif")

Who's Ahead Now is live at [www.whosaheadnow.com](http://www.whosaheadnow.com). Please contribute to the [source code](https://github.com/ktravers/whos-ahead-now); there's a lot of potential features to add, plus plenty of code clean-up needed as well. Thanks to [HuffPost's Pollster API](http://elections.huffingtonpost.com/pollster/api) for the easily accessible and excellent polling data.
