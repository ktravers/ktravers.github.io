---
layout: post
title: Monkey Patching truncate_html
tags: [ruby]
---

The [`truncate_html` gem](https://github.com/hgmnz/truncate_html) is a really useful tool for clipping off a string of html at a designated point. It has some nice [customizeable config options](https://github.com/hgmnz/truncate_html#example), and best of all, zero third party dependencies. Per its docs, it even does the unthinkable:

> This library...[parses HTML using regular expressions](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454).

Today I discovered I could extend its usefulness even further with a small monkey patch. Now, before the [haters](http://devblog.avdi.org/2008/02/23/why-monkeypatching-is-destroying-ruby/) come out in full force, YES - I understand that monkey patching is dangerous and needs to be handled with deft and delicacy. That said, the small size and limited dependencies of this gem make it a prime candidate for [practicing safe patching, Justin-Weiss style](http://www.justinweiss.com/blog/2015/01/20/3-ways-to-monkey-patch-without-making-a-mess/).

Moving right along...

### Desired functionality:
### 1. Multiple break tokens

Per [`truncate_html`'s docs](https://github.com/hgmnz/truncate_html#example), you can truncate your html after a after a certain number of characters (`options[:length]`) or at a designated piece of content, like `<!-- break -->` (`options[:break_token]`).

Setting a single `break_token` is great if I've created the HTML and set my own unique `break_token` wherever necessary. But what about situations where I want to pass in any old HTML and truncate based on the content? If I want to cut the string before it hits a video for example, I'll need two `break_tokens` at minimum - `video` and `iframe`. I could do this with a couple iterations over the same HTML string, but it'd be way better to just be able to set multiple `break_tokens` right there in the options hash.

### 2. More flexible token matching

Playing around with different tokens, I noticed that if I set `break_token` to an html element, like `<img>` or `img`, for example, the string wouldn't be truncated as I intended if that element had any attributes, like `class`, `id`, or `src`. I had to match the full tag content exactly.

To illustrate:

```ruby
html_string = "<h1 class='title'>Cats!</h1><img src='/img/cat.gif'><p>Cats are cute.</p>"

truncate_html(html_string, break_token: 'h1')
# => "<h1 class='title'>Cats!</h1><img src='/img/cat.gif'><p>Cats are cute.</p>"
# wat?

truncate_html(html_string, break_token: '<img>')
# => "<h1 class='title'>Cats!</h1><img src='/img/cat.gif'><p>Cats are cute.</p>"
# double wat?

truncate_html(html_string, break_token: "<img src='/img/cat.gif'>")
# => "<h1 class='title'>Cats!</h1>"

truncate_html(html_string, break_token: "<p>")
# => "<h1 class='title'>Cats!</h1><img src='/img/cat.gif'>"
```

I went into the [source](https://github.com/hgmnz/truncate_html/blob/master/lib/truncate_html/html_string.rb) to see what was going on, and turns out this behavior was completely by design. When your html string is passed into HtmlString, it's spliced into an array of `html_tokens`, each one an open tag, closed tag, comment or plain text.

For example:

```ruby
html_string = "<h1 class='title'>Cats!</h1><img src='/img/cat.gif'><p>Cats are cute.</p>"

HtmlString.new(html).html_tokens
# => [ "<h1 class='title'>",
#     "Cats!",
#     "</h1>",
#     "<img src='/img/cat.gif'>",
#     "<p>",
#     "Cats",
#     " ",
#     "are",
#     " ",
#     "cute.",
#     "</p>" ]
```

Each one of these `html_tokens` is matched against your `break_token`, and only a perfect match will trigger truncate. I'd like to make this a little more flexible.

Objectives identified, it's time to dive into the actual patch. Fortunately for me, I'm not the only person who wants multiple `break_tokens`. As of this writing, there's an [open PR on the `truncate_html` repo](https://github.com/hgmnz/truncate_html/pull/53) that takes care of objective #1. Objective #2 requires a small adjustment to the logic in `truncate_token?(token)`. Let's implement both fixes below.


### The Monkey Patch:
### Option 1: Reopen Class and Override Methods (blunt)

My first solution was to just dive in, open up the necessary classes, and overwrite away - delicacy be damned. Pros of this approach are that I only had to change one file, `config/initializers/truncate_html.rb`. We'll explore the cons below.

```ruby
# config/initializers/truncate_html.rb
class TruncateHtml::Configuration
  attr_accessor :break_tokens
end

class TruncateHtml::HtmlTruncator
  def initialize(original_html, options = {})
    @original_html   = original_html
    length           = options[:length]        || TruncateHtml.configuration.length
    @omission        = options[:omission]      || TruncateHtml.configuration.omission
    @word_boundary   = options[:word_boundary] || TruncateHtml.configuration.word_boundary
    @break_token     = options[:break_token]   || TruncateHtml.configuration.break_token  || nil
    @break_tokens    = options[:break_tokens]  || TruncateHtml.configuration.break_tokens || []
    @break_tokens    << @break_token if @break_token
    @chars_remaining = length - @omission.length
    @open_tags, @closing_tags, @truncated_html = [], [], ['']
  end

  private

  def truncate_token?(token)
    @break_tokens.any? do |break_token|
      token.include?(break_token)
    end
  end
end
```

While this approach worked, it's a little blunt.

![Gratuitous Blunt]({{ site.baseurl }}/images/posts/emily-blunt.gif "Gratuitous Blunt")

As [Justin Weiss cautions](http://www.justinweiss.com/blog/2015/01/20/3-ways-to-monkey-patch-without-making-a-mess), there are a number of potential problems with punching your patch in this way:

> **1. If two libraries monkey-patch the same method, you won’t be able to tell.**  
>    - The first monkey-patch will get overwritten and disappear forever.  
>
> **2. If there’s an error, it’ll look like the error happened inside [TruncateHtml].**  
>    - While technically true, it’s not that helpful.  
>
> **3. It’s harder to turn off your monkey patches.**  
>    - You have to either comment out your entire patch, or skip requiring your monkey patch file if you want to run code without it.  
>    - If you, say, forgot to require [truncate_html] before running this monkey patch, you’ll accidentally redefine [the class] instead of patching it.  

A much safer, surgical solution is to put your patch in a module.


### Option 2: Put Patches Inside Modules (not blunt)

Following [Justin's](http://www.justinweiss.com/blog/2015/01/20/3-ways-to-monkey-patch-without-making-a-mess/#) and many [other Rubyists recommendations](http://stackoverflow.com/questions/580314/overriding-a-module-method-from-a-gem-in-rails), I refactored my patch so that it was safely stowed inside namespaced modules.

I added individual modules for each monkey-patched class, organized into folders like so: `lib/gem_extensions/:gem_name/:class.rb`. Note: reader recommendations welcome on better ways to organize this.

Finally, I included each module in the `truncate_html` initializer using #send.

```ruby
# lib/gem_extensions/truncate_html/configuration.rb
module GemExtensions
  module TruncateHtml
    module Configuration
      attr_accessor :break_tokens
    end
  end
end

# lib/gem_extensions/truncate_html/html_truncator.rb
module GemExtensions
  module TruncateHtml
    module HtmlTruncator
      def initialize(original_html, options = {})
        @original_html   = original_html
        length           = options[:length]        || TruncateHtml.configuration.length
        @omission        = options[:omission]      || TruncateHtml.configuration.omission
        @word_boundary   = options[:word_boundary] || TruncateHtml.configuration.word_boundary
        @break_token     = options[:break_token]   || TruncateHtml.configuration.break_token  || nil
        @break_tokens    = options[:break_tokens]  || TruncateHtml.configuration.break_tokens || []
        @break_tokens    << @break_token if @break_token
        @chars_remaining = length - @omission.length
        @open_tags, @closing_tags, @truncated_html = [], [], ['']
      end

      private

      def truncate_token?(token)
        @break_tokens.any? do |break_token|
          token.include?(break_token)
        end
      end
    end
  end
end

# config/initializers/truncate_html.rb
TruncateHtml.send(:include, GemExtensions::TruncateHtml::Configuration)
TruncateHtml.send(:include, GemExtensions::TruncateHtml::HtmlTruncator)
```

Benefits of this approach include:  

1. Better error tracing - now clear when errors originate in module vs gem  

2. Easy to include / exclude modules - just comment out as needed in the initializer  

3. Clearer intentions - easy for new developers to recognize what these modules do and why  


### Option 3: Fork original repo and update Gemfile

Thanks to the wonders of Bundler (and [this excellent blog post](http://www.rubyinside.com/the-end-of-monkeypatching-4570.html)), we can actually bring in all this new functionality without adding any new files to our project.

Simply fork the original repo, push up your changes, then point your Gemfile to your fork.

```ruby
# Gemfile
# Change from this:
gem 'truncate_html'

# To this:
gem 'truncate_html', :git => 'git://github.com/ktravers/truncate_html', :branch => 'master'
```

Run `bundle` and you're done.


Resources:  
1. [The End of Monkeypatching - Xavier Shay](http://www.rubyinside.com/the-end-of-monkeypatching-4570.html)  
2. [3 Ways to Monkey Patch Without Making a Mess](http://www.justinweiss.com/blog/2015/01/20/3-ways-to-monkey-patch-without-making-a-mess/#)  
3. [Stack Overflow - Overriding a module method from a gem in rails](http://stackoverflow.com/questions/580314/overriding-a-module-method-from-a-gem-in-rails)  
4. [truncate_html gem](https://github.com/hgmnz/truncate_html)  
