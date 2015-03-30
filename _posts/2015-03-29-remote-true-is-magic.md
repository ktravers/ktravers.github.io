---
layout: post
title: Remote True is Magic
---

I had a really fun time this weekend building a [small side project](http://www.amiruby.com/) with [Sophie DeBenedetto](https://github.com/SophieDeBenedetto), [Jeremy Sklarsky](https://github.com/jeremysklarsky), and [Rachel Nackman](https://github.com/rnackman). We learned a lot of very cool things that you can do with Rails + jQuery, but the coolest was easily `remote: true`. This one little option is crazy powerful. Case in point: by adding `remote: true` to our `Search` form, our `searches.js` file went from this:

```javascript
$(function(){submitListener();})
function submitListener(){$('#new_search').on("submit", submitSearch)}

function submitSearch(e){
  e.preventDefault();
  var url = $(this).attr('action')
  var $form =  $("form#new_search");
  $.ajax(url, {
    method: "POST",
    url: url,
    "data": $form.serialize(),
    dataType: "script",
    "complete": function(response){}
  });
}
```

to this:

```

```

That's right. Adding `remote: true` allowed us to nix our entire jQuery listener + ajax request for that form, because all that functionality is contained within `remote: true`. 

<div align="center">
![Taylor Swift Mind Blown]({{ site.baseurl }}/assets/mind-blown-taylor.gif "Taylor Swift Mind Blown")
</div>

So how's it work? Maybe you're thinking this:

![Remote True is MAGIC]({{ site.baseurl }}/assets/remote-true-is-magic.png "Remote True is MAGIC")


Let's demystify it a bit. The Rails documentation puts things pretty simply:

> Rails provides a bunch of view helper methods written in Ruby to assist you in generating HTML [including Ajax helpers]. ...The Rails "Ajax helpers" are in two parts: the JavaScript half and the Ruby half.

> `rails.js` provides the JavaScript half, and the regular Ruby view helpers add appropriate tags to your DOM. The CoffeeScript in `rails.js` then listens for these attributes and attaches appropriate handlers.

To recap, when you use `remote: true` inside a view helper method, like `form_for` or `link_to`, Rails automatically **adds a jQuery listener to that object** (the form or the link) and **executes the appropriate action** (such as `e.preventDefault();` and `e.stopPropogation();`, as two examples). With that basic functionality taken care of, you're then free to add your own jQuery actions as needed within a `.js.erb` view file, after making sure your controller will `respond_to` both `format.js` and `format.html`. 

That explanation was pretty abstract, so let's look at a concrete example of how to implement `remote: true` inside some actual working code. I'll step through pieces of our code below, and you can always [check out the repo here](https://github.com/jeremysklarsky/AmIRuby) for the full source.

#### Step 1: Add `remote: true` to your form. 

```html
<!-- index.html.erb -->

<h1>Am I Ruby?</h1>
<%= form_for(@search, remote: true) do |f| %>
  <%= f.text_field :keyword, id: 'keyword' %><br><br>
  <%=f.submit "Search", class: "search-btn"%>
<% end %>
<section id="result"></section>
```

Originally, our `form_for` method took a single argument, `@search`. We simply added the additional option `remote: true` to that argument. Now our form submits data (formerly expressed as `data: $form.serialize()` inside the ajax method) to the server without refreshing the page. However, you won't see anything happen in your browser yet, because our contorller doesn't yet know how exactly to `respond_to` the successful ajax request. We'll need to make some adjuments there next.

#### Step 2: Update your controller action to `respond_to` `format.js`.

```ruby
class SearchesController < ApplicationController

  def create
    @result = Search.new.am_i_ruby(params[:search][:keyword])    
    respond_to do |format|
      format.html
      format.js 
    end
  end
end
```

Before "ajaxifying" our code, our `create` action only had one line of code setting the `@result` variable. _Note:_ We could also have explicitly told it to render a specific view, but since by default, the `create` action will render the `create.html.erb` view, we had left that line out.

```ruby
# non-ajax'd SearchController create action
def create
  @result = Search.new.am_i_ruby(search_params)
end
```

Now that we're using `remote: true`, we don't want to render a view; we want to execute a jQuery function of our choosing. So we replace the `render` command and with `respond_to` instructions for each format we want our controller to serve. Here, our controller can serve html AND Javascript responses to the ajax success function. 

#### Step 3: Add `create.js.erb` file

In your `create.js.erb` view file, you can add whatever actions you want to happen upon a successful ajax request. In our case, we wanted to display the result of our search form (which was querying BuiltWith.com and returning "yes" or "no", depending on whether or not the domain we sent it included Ruby code in its stack).

```javascript
$('#result').hide().html('<%= j @result %>').fadeIn(250);
```

Our jQuery command is doing the following:
- selecting the div with "id=result"  
- upon document ready, hiding that div  
- listening for successful form submission (performed behind-the-scenes by `remote: true`)  
- upon successful ajax request, inserting `@result` into DOM html inside `div#result` with a nice 250ms count fade-in effect  

Check out the code in action here: [www.amiruby.com](http://www.amiruby.com/)

### More resources:
1. [Railscasts #136 - jQuery & Ajax](http://railscasts.com/episodes/136-jquery-ajax-revised)  
2. [Railscasts #205 - Unobtrusive Javascript](http://railscasts.com/episodes/205-unobtrusive-javascript)  
3. [Alfajango - Rails Remote Links and Forms](http://www.alfajango.com/blog/rails-3-remote-links-and-forms/)  
4. [CodeBeerStartups - Ajaxify your Site With Remote True](http://www.codebeerstartups.com/2012/12/ajaxify-your-site-with-remote-true)  
5. [(Flatiron alum) Koren Leslie Cohen - Remote True in Rails Forms](http://www.korenlc.com/remote-true-in-rails-forms/)
