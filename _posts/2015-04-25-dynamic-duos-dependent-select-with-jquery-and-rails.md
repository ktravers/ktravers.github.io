---
layout: post
title: Dynamic Duos - Dependent Select Menus with JQuery and Rails
tags: ['rails', 'ruby', 'jquery']
---

The first time I tried to apply "dynamic selection" within a Rails form - where selecting an option in one select field dynamically updates the menu options in a second select field, I had a surprisingly hard time with it. I'd watched the [Railscast](http://railscasts.com/episodes/88-dynamic-select-menus-revised) and [read](http://pullmonkey.com/2012/08/11/dynamic-select-boxes-ruby-on-rails-3/) [multiple](https://kernelgarden.wordpress.com/2014/02/26/dynamic-select-boxes-in-rails-4/) [tutorials](http://www.falsepositives.com/index.php/2010/05/28/building-a-casscading-drop-down-selection-list-for-ruby-on-rails-with-jquery-ajax/), but just couldn't crack the code.

Problem was, I was trying to use a collection of _model attributes_ in the first [`collection_select`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_select) menu and a collection of _model instances_ in the second menu, organized using the [`grouped_collection_select`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-grouped_collection_select) helper. What I learned after much trial, error and a deep dive into the [documentation](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-grouped_collection_select), was that **the menu options should both be collections of model instances, where the models are associated through a join table**. <-- _tldr: central lesson of this post._

I could have benefitted from more explicit discussion of those mechanisms, one with a slightly more complex example than countries and states. Towards that end, let's walk through the steps I took to apply this solution in my Rails application, **[Parkster](http://www.parksternyc.com/)**.

SECTIONS:  
1. [Correcting the schema](#update-schema)  
2. [Updating the seeds file](#update-seeds)  
3. [Adding appropriate ActiveRecord associations](#update-associations) to models  
4. [Updating the form partial](#update-form)  
5. [Adding jQuery](#add-jquery), a nice alternative to [Railscasts' CoffeeScript solution](http://railscasts.com/episodes/88-dynamic-select-menus-revised)

### Pre-Refactor Setup
All code here is from my Rails app **[Parkster](http://www.parksternyc.com/)**, a [Meetup](http://www.meetup.com/)-esque app that allows users to organize games in NYC parks with friends. You can check out the full source code [here](https://github.com/ktravers/park_friends).

Before the refactor, my schema was structured as follows:

[ ![Parkster schema before]({{ site.baseurl }}/images/posts/schema-before.png "Parkster schema before") ](https://github.com/voormedia/rails-erd "Generated with rails-erd gem")

You'll notice that Parks have an `#activity` attribute (ex. "Baseball Field", "Basketball Court"), and Games have a `#game_category` attribute (ex. "Baseball", "Basketball"). Hopefully some readers' smell test sensors are pinging here already...

The Game form lives in its own partial that's rendered from both the `create.html.erb` and `edit.html.erb`. I originally utilized the [SimpleForm gem](https://github.com/plataformatec/simple_form) with ActionView [form_for helper](http://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_for).

```html
<div class="game-form">
  <%= simple_form_for(@game) do |f| %>
    <div class="input-group input-group-lg">
      <span class="input-group-addon"><%= f.label "Activity" %></span>
      <%= f.select :game_category, collection: @game.game_categories.keys, class: "form-control", placeholder: "Baseball" %>
    </div>

    <div class="input-group input-group-lg">
      <span class="input-group-addon"><%= f.label :location %></span>
      <%= f.select :park_id, label: false, collection: Park.all.order("name"), class: "form-control" %>
    </div>

    <%= f.submit %>
  <% end %>
</div>
```

Finally, the seeds file made a simple JSON call to [NYC Open Data's](https://data.cityofnewyork.us/Recreation/Directory-of-Parks-Disability-Accessibility-Facili/e4ej-j6hn?) [Socrata API](http://dev.socrata.com/foundry/#/data.cityofnewyork.us/e4ej-j6hn), creating Park instances and persisting them to the database.

```ruby
# seeds.rb
require 'open-uri'
json = JSON.load(open("https://data.cityofnewyork.us/resource/e4ej-j6hn.json"))
json.each do |park|
  Park.create(
    name: park['name'],
    location: park['location'],
    facility: park['type']
  ) unless park['type'] == 'Bathroom' # filter out parks without game facilities
end
```

### <a id="update-schema"></a>Step 1: Update schema

We noticed above that there's a relationship between Park `#activity` and Game `#game_category`, so first thing is to extract this connection into its own model and associated join table.

Run two commands from the terminal:  
 1. `rails g migration CreateActivities name:string`  
 2. `rails g migration CreateActivityParks park_id:integer activity_id:integer`  

### <a id="update-seeds"></a>Step 2: Update seeds file

Activities and ActivityParks are derived from the Park #facility attribute, so we can efficiently create and persist instances of all three models on the fly in our seeds file. Given the limited number of unique activities/facilities, I used a simple switch statement on JSON `park['type']` data.

```ruby
# seeds.rb
require 'open-uri'
json = JSON.load(open("https://data.cityofnewyork.us/resource/e4ej-j6hn.json"))
json.each do |park|
  park = Park.create(
    name: park['name'],
    location: park['location'],
    facility: park['type']
  ) unless park['type'] == 'Bathroom' # filter out parks without game facilities

  park_activity = ''
  park_facility = Park.all.last.facility # most recently-created Park's facility

  case park_facility
  when 'Ice Skating Rinks'
    park_activity = 'Ice Skating'
  when 'Bocce Courts'
    park_activity = 'Bocce'
  when 'Basketball Courts'
    park_activity = 'Basketball'
  when 'Tennis Courts'
    park_activity = 'Tennis'
  when 'Football Fields'
    park_activity = 'Football'
  when 'Baseball Fields'
    park_activity = 'Baseball'
  when ''
    park_activity = 'Other'
  end

  activity = Activity.find_or_create_by(:name => park_activity)
  ActivityPark.create(:park_id => park.id, :activity_id => activity.id)
end
```

### <a id="update-associations"></a>Step 3: Add ActiveRecord associations

The updated model associations are fairly straight-forward.

A Park has one Activity, through ActivityParks.

```ruby
class Park < ActiveRecord::Base
  has_one :activity_park
  has_one :activity, through: :activity_park
  has_many :games
  has_many :reservations, through: :games
end
```

An Activity has many Parks, through (many) ActivityParks.

```ruby
class Activity < ActiveRecord::Base
  has_many :activity_parks
  has_many :parks, through: :activity_parks
end
```

An ActivityPark belongs to one Park and one Activity.

```ruby
class ActivityPark < ActiveRecord::Base
  belongs_to :activity
  belongs_to :park
end
```

Our migrations, associations, and seeds file are all properly updated, so now's a good time to run `rake db:reset` (shortcut for `db:drop, db:create, db:schema:load, db:seed`). Now we have a schema we can work with:

[ ![Parkster schema after]({{ site.baseurl }}/images/posts/schema-after.png "Parkster schema after") ](https://github.com/voormedia/rails-erd "Generated with rails-erd gem")

### <a id="update-form"></a>Step 4: Update form partial

At this point, it was easiest for me to remove SimpleForm and use the built-in ActiveView form_for helper instead. The first select menu should use the [`collection_select`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-collection_select) FormOptionsHelper:

```ruby
# ActionView::Helpers::FormOptionsHelper#collection_select
collection_select(
  object, method, collection, value_method, text_method, options={}, html_options={}
)
```

We want this menu to list each Activity by name, and send the user's selection to the GamesController's `#create` or `#update` action as part of `game_params`, specifically `params['game']['game_category']`. So we'll set the method/attribute as `#game_category`, collection as `Activity.order(:name)`, and `value_method` as `#name`. I included the Bootstrap "form-control" class as part of the `html_options` hash for styling purposes.

The second select menu options need to be grouped by Park #activity, so we'll use the [`grouped_collection_select`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-grouped_collection_select) FormOptionsHelper:

```ruby
# ActionView::Helpers::FormOptionsHelper#grouped_collection_select
grouped_collection_select(
  object, method, collection, group_method, group_label_method, option_key_method, option_value_method, options={}, html_options={}
)
```

This menu should list each Park by name, grouped by associated Activity, and send the user's selection to the GamesController's `#create` or `#update` action as part of `game_params`, specifically `params['game']['park_id']`. We'll set the method/attribute as `park_id`, collection as `Activity.all`, and group by group_method `#parks` (i.e. all the parks that are associated with that instance of `Activity`).

```html
<div class="game-form">
  <%= form_for(@game) do |f| %>
    <div class="input-group input-group-lg">
      <span class="input-group-addon"><%= f.label "Activity" %></span>
      <%= f.collection_select :game_category, Activity.order(:name), :name, :name, include_blank: 'Select an activity...', class: "form-control" %>
    </div>

    <div class="input-group input-group-lg">
      <span class="input-group-addon"><%= f.label :location %></span>
      <%= f.grouped_collection_select :park_id, Activity.all, :parks, :name, :id, :name, include_blank: 'Select a park...', class: "form-control" %>
    </div>

    <%= f.submit %>
  <% end %>
</div>
```

### <a id="add-jquery"></a>Final Step: Add jQuery

On document load, we want to add a listener to the first select menu, which `form_for` has given the helpful id of `#game_game_category`. If that menu's value changes (e.g. if a user makes a selection), it'll trigger a function that grabs the selected option's value and passes it to the second select menu, `#game_park_id`, as a filter, so the menu's `optgroup` automatically filters to show just the Parks associated with the user's selection.

```javascript
$(function(){
  filterParksList();
})

function filterParksList(){
  var parks = $('#game_park_id').html();

  $('#game_game_category').change(function(){
    var selectedGameCategory = $('#game_game_category :selected').text();
    var optgroup = "optgroup[label='"+ selectedGameCategory + "']";
    var parkOptions = $(parks).filter(optgroup).html();

    if(selectedGameCategory != 'Other'){
      $('#game_park_id').html(parkOptions);
    }
  });
}
```

I made a conscious decision to leave the "Other" Activity category available to users, and in the case when "Other" is selected, we'd want users to be able to choose from all NYC parks. That's why the jQuery doesn't filter `#game_park_id`'s `optgroup` for that case.

---

That's the full refactor. Your select menus are now working in tandem, filtering on change like a truly dynamic duo. Hope it was helpful - let me know in the comments below!


#### More helpful resources:

1. [Railscasts ep. 88: Dynamic Select Menus (Revised)](http://railscasts.com/episodes/88-dynamic-select-menus-revised)  
2. [Rails documentation: `grouped_collection_select`](http://api.rubyonrails.org/classes/ActionView/Helpers/FormOptionsHelper.html#method-i-grouped_collection_select)  
3. [Pull Monkey blog - Dynamic Select Boxes in Rails 3](http://pullmonkey.com/2012/08/11/dynamic-select-boxes-ruby-on-rails-3/)  
4. [Kernel Garden blog - Dynamic Select Boxes in Rails 4](https://kernelgarden.wordpress.com/2014/02/26/dynamic-select-boxes-in-rails-4/)  
5. [False Positives blog - Building Cascading Drop Down Selection List for Rails with JQuery/Ajax](http://www.falsepositives.com/index.php/2010/05/28/building-a-casscading-drop-down-selection-list-for-ruby-on-rails-with-jquery-ajax/)  
