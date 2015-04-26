---
layout: post
title: Rendering from a Model with ActionView::Base.new => WAT?
---

I'm a firm believer in the "make it work" philosophy - solve problems first, then refactor. That said, my team may have gotten a little too creative making our [last project](http://www.approvablefeast.com/) work. Just take a look at this gnarly method we cooked up:

```ruby
class Recipe < ActiveRecord::Base
  def recipe_card
    ActionView::Base.new(
      Rails.configuration.paths['app/views']).render(
      :partial => 'menus/recipe', :format => :txt,
      :locals => { :recipe => self })
      ###🙊🚷 W A T ☠😿###
  end
end
```

Yep. We instantiated a new instance of ActionView::Base in a model. And used it to render a view outside a controller. [WAT?!](https://www.destroyallsoftware.com/talks/wat)

Before I go any further, let's take care of the necessaries:  
- Yes, we know what we did was wrong.  
- Yes, we are deeply sorry for our crimes against MVC.  
- Yes, we'll refactor and never repeat this grievous offense again.  

But the thing is, it worked. And I'd like to explore how and why before ~~purging this from our git history~~ moving forward.

## THE PROBLEM

The problem we were trying to solve was relatively simple. In our Menu views, we'd created a partial `_menu` that contains multiple [`collection_check_boxes`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes) fields. Each checkbox renders another partial `_recipe`, which serves up an image and a link. The end result is a row of recipes cards, where each card has an image, a link, and a checkbox that users can tick to select that recipe for their final menu:

![Approvable Feast recipe cards]({{ site.baseurl }}/assets/recipe-cards.png "Approvable Feast recipe cards")

After reviewing the documentation on [`collection_check_boxes`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes), we thought we'd be able to pass the `render` method inside the `&block` argument.

```ruby
collection_check_boxes(object, method, collection, value_method, text_method, options = {}, html_options = {}, &block)
```

Unfortunately, no matter what we tried, we couldn't get that `_recipe` partial to render. We tried passing HTML options to the `label` and `check_box` builder methods. Nothing. We tried utilizing the "special" `object`, `text`, and `value` methods - no dice. 

Ultimately, we kept circling back to the `text_method` argument. In most of the examples we found online, `text_method` called a method defined in the form's associated model. This got us thinking - could we write a method in the Recipe model that would render the `_recipe` partial? That way we could pass it as an argument into [`collection_check_boxes`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes) and boom, problem solved.

## THE WAT SOLUTION

Turns out with a little hackery, you can render a view from a model. Let's follow the pass-and-catch below. 

☠ Start in the menu partial, where [`collection_check_boxes`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes) `:recipe_card` argument points us to the Recipe model's method `#recipe_card`.

```html
<!-- menu partial -->
<%= form_for @dinner.menu, method: "PATCH", remote: true do |f|%>
  <div id="appetizers">
    <h3>Appetizers</h3>
    <div class="recipe-card horizontal-scroll">
      <%= f.collection_check_boxes(:recipes, @dinner.menu.appetizers, :id, :recipe_card) %>
    </div>
  </div>
  <%= f.submit "Set Menu", class: "btn btn-primary" %>
<% end %>
```

☠ Inside Recipe, `#recipe_card` instantiates a new instance of ActionView::Base and calls `#render` on it, passing in the recipe partial, `:format`, and local variables - in this case, the current instance of Recipe.

```ruby
class Recipe < ActiveRecord::Base
  def recipe_card
    ActionView::Base.new(
      Rails.configuration.paths['app/views']).render(
      :partial => 'menus/recipe', :format => :txt,
      :locals => { :recipe => self })
  end
end
```

☠ Nothing strange happening in the recipe partial. We have access to an instance of Recipe, thanks to the locals we passed in, so the partial serves up the recipe's image, `short_name`, and link, blissfully unaware of the oddness that's allowing it to do so.

```html
<!-- recipe partial -->
<div class="recipe-card-partial">
  <div class="recipe-card-partial-image">
    <img src="<%= recipe.image_upload.url %>" class="img-thumbnail"><br>
    <%= link_to recipe.short_name, "/recipes/#{recipe.id}"%>
  </div>
</div>
```

## THE WHY

Let's break down exactly what's happening in that `#recipe_card` method. 

```ruby
def recipe_card
  ActionView::Base.new(
    Rails.configuration.paths['app/views']).render(
    :partial => 'menus/recipe', :format => :txt,
    :locals => { :recipe => self })
end
```

First up, let's inspect [`ActionView::Base.new(args)`](https://github.com/rails/rails/blob/700ec897f97c60016ad748236bf3a49ef15a20de/actionview/lib/action_view/base.rb). If you check the [source code](https://github.com/rails/rails/blob/700ec897f97c60016ad748236bf3a49ef15a20de/actionview/lib/action_view/base.rb), you'll see that the first argument passed to `ActionView::Base#initialize` is `context` [(line 185)](https://github.com/rails/rails/blob/700ec897f97c60016ad748236bf3a49ef15a20de/actionview/lib/action_view/base.rb#L185). The context we're passing here is [`Rails.configuration.paths['app/views']`](https://github.com/rails/rails/blob/f295c2fb364e2b6b5d73073c2a3287bbbe7c81fa/railties/lib/rails/application/configuration.rb#L79), which selects the `app/views` path object from our application. Anyone curious about what that path object looks like can check it out [here](https://gist.githubusercontent.com/ktravers/295bebf2ed87c89aa54a/raw/2dc2c5ba854541f8e2da765bd2ac951850289288/rails-config-paths-app-views) (spoiler alert: it's a doozy). _Note:_ Since we're instantiating outside the controller, we have to explicitly configure our view paths, otherwise our new ActionView instance won't have access to them.

Moving on, as `context` travels through the `#initialize` method, it's eventually passed into [`ActionView::Renderer`](http://api.rubyonrails.org/classes/ActionView/Renderer.html) (in [line 195](https://github.com/rails/rails/blob/700ec897f97c60016ad748236bf3a49ef15a20de/actionview/lib/action_view/base.rb#L195), in our case), which parses it into the ultimate return value from `#initialize`. We then call `#render` on that return value, passing it :partials and :locals arguments. That `#render` method is the ["main render entry point shared by ActionView and ActionController"](http://api.rubyonrails.org/classes/ActionView/Renderer.html#method-i-render), and it taps [`ActionView::PartialRenderer`](http://api.rubyonrails.org/classes/ActionView/PartialRenderer.html) to take care of the rest of the work.

Under normal circumstances, [controllers instantiate new instances of the ActionView::Base class](http://api.rubyonrails.org/classes/ActionController/Base.html#class-ActionController::Base-label-Renders), so the process described above would be triggered by calling `#render` in the controller. However, by initializing ActionView::Base in the model, we get to define our own context and bypass the normal ActionController > ActionView > render template flow. That allows our model method `#recipe_card` to serve up an instance of that view whenever and wherever it's called. Wild.

## WAT DID WE LEARN

Main takeaway: while it's not too hard to override Rails' MVC conventions, it's almost always a bad idea. First off, it introduces unnecessary complexity into your code base. Look, it took me 3 paragraphs to break down what was happening in that `#recipe_card` method. Imagine if we'd pulled this same trick other places in our code. Good luck to new team adds trying to grok / refactor that...

Also, if you find yourself trying to override Rails built-in conventions, that's probably a good sign you're approaching your problem the wrong way. I'm not [José Valim](https://github.com/josevalim); there's probably a simpler solution. Likely somewhere in the [`collection_check_boxes`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_check_boxes) [source code](https://github.com/rails/rails/blob/71c7fd101324046995d8f7e51e78475c0e37ec1a/actionview/lib/action_view/helpers/form_options_helper.rb#L710), which is where I'm heading next...

---

_For the record:_  
We're not the only rogues who've tried to pull this caper. I've linked to a few other offenders below, including the [render_anywhere gem](https://github.com/yappbox/render_anywhere), which we briefly considering using. Our solution seems like a similar (but slightly simpler) version of the gem's. For example, the gem sets up a dummy controller, [RenderingController](https://github.com/yappbox/render_anywhere/blob/a1b6d6e1a61e07e5e956ea645f5df49c3fb28a1c/lib/render_anywhere/rendering_controller.rb), and uses it to override Rails' built-in `render` method. Since we only needed this unconventional behavior in one place, we opted to write our own weird method, rather than utilize a third-party library.

Thanks for reading, and feel free to add feedback / corrections in the comments below!

#### More resources:
1. [Alvin Liang](https://github.com/aliang) => [Render from model in Rails 3](https://gist.github.com/aliang/1022384)  
2. [render_anywhere gem](https://github.com/yappbox/render_anywhere) from [Luke Melia](https://github.com/lukemelia) of [YappLabs](https://www.yapp.us/)  
3. [The Devel! blog](http://blog.choonkeat.com/) => [Rails: Calling render() outside your Controllers](http://blog.choonkeat.com/weblog/2006/08/rails-calling-r.html)

