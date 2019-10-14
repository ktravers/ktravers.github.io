---
layout: post
title: AcademyAwardable Polymorphic Associations
tags: ['rails', 'ruby']
---

If you ever find yourself in a Rails-situation where you need one model to belong to multiple other models, consider using an [ActiveRecord polymorphic association](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#module-ActiveRecord::Associations::ClassMethods-label-Polymorphic+Associations). Don't let the multisyllabic name fool you; [polymorphic associations](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations) aren't as complex to build as they might seem. You can do it.

![Meryl Streep Can Do It]({{ site.baseurl }}/images/posts/meryl.jpg "Meryl Streep Can Do It")

Let's consider a completely relatable and engaging example: [The Academy Awards](http://oscar.go.com/). Maybe you're building an Oscar ballot app for a [family member](http://www.indiewire.com/author/ben-travers) who's particularly obsessed with this time-honored, hallowed awards show. Your app at minimum would need to contain Actor, Movie, Director, and Vote models (plus all those [technical award categories](http://www.oscars.org/sci-tech), but we're going to ignore them for now - just like the Academy). We'll need some savvy schema design to keep things simple and DRY.

Let's start by trying to set up our schema the old-fashioned, non-polymorphic way first. Actor, Movie, and Director would all `have_many` Votes, which means Vote would `belong_to` Actor, Movie, and Director. Here's how that would look in our `CreateVotes` migration:

```ruby
class CreateVotes < ActiveRecord::Migration
  def change
    create_table :votes do |t|
      t.integer  :actor_id
      t.integer  :movie_id
      t.integer  :director_id
      t.timestamps null: false
    end

    add_index :votes, :actor_id
    add_index :votes, :movie_id
    add_index :votes, :director_id
  end
end
```

And in our Vote model:

```ruby
class Vote < ActiveRecord::Base
  belongs_to :actor
  belongs_to :movie
  belongs_to :director
end
```

Man, I haven't seen that much gratuitous repetition since Chris Nolan's [Memento](http://www.imdb.com/title/tt0209144/). Let's DRY this up a bit with a polymorphic association.

First step, we need to ask ourselves what our non-Vote models have in common. Or better, what common trait do they all share? In this case, they're all able to be voted on - that is, they're `Voteable`. Our polymorphic association is born.

We can update our `CreateVotes` migration as follows:

```ruby
class CreateVotes < ActiveRecord::Migration
  def change
    create_table :votes do |t|
      t.integer  :voteable_id
      t.string   :voteable_type
      t.timestamps null: false
    end

    add_index :votes, :voteable_id
  end
end
```

_Note:_ You can simplify the migration even further by using `t.references`; however, be careful here you're not losing clarity along with a few lines of code.

```ruby
class CreateVotes < ActiveRecord::Migration
  def change
    create_table :votes do |t|
      t.references :voteable, polymorphic: true, index: true
      t.timestamps null: false
    end
  end
end
```

Notice that instead of adding explicit foreign keys for Actor, Movie, and Director, we're instead adding a single `imageable_id` foreign key, which Rails will automagically couple with the `imageable_type` field to reference the correct corresponding model.

Speaking of models, ours look much better now with our new polymorphic-association-enabled, DRY-er declarations:

```ruby
class Vote < ActiveRecord::Base
  belongs_to :voteable, polymorphic: true
end

class Actor < ActiveRecord::Base
  has_many :votes, as: :voteable
  # has_all_the :votes, as: :MerylStreep
end

class Movie < ActiveRecord::Base
  has_many :votes, as: :voteable
end

class Director < ActiveRecord::Base
  has_many :votes, as: :voteable
end
```

Thanks to our polymorphic `belongs_to: :voteable` declaration, we now have access to all kinds of fun methods on our models. We can call `@actor.votes`, `@movie.votes` and `@director.votes`, as expected. To retrieve a Vote object's parent, we can call `@vote.voteable`, which returns its corresponding `Actor`, `Movie`, or `Director`. Throw a User class in there - plus some simple routes, controllers, and views - and we have the makings of a pretty functional Oscar ballot app - - - but that's the topic for another (future) post. In the meantime, bravo. You've built an award-worthy polymorphic association.

![Meryl Streep Loves It]({{ site.baseurl }}/images/posts/meryl.gif "Meryl Streep Loves It")

## Summary (tldr) <a href="#tldr"></a>

### 1. Do you have one model that `belongs_to` many other models?
- Does this one model only exist in relation to the other models?   
  Examples: Comments and Posts; Likes and Tweets  
- Think about using polymorphic association instead of multiple foreign keys / `belongs_to` declarations  
        
### 2. Ask yourself what these models have in common
- Describe that commonality in a single trait, like `Commentable` or `Likeable`  
- Use this shared trait to name the polymorphic association  

### 3. Build model migrations and associations
- Model migrations: use `awardable_id` and `awardable_type` columns  
- Model associations: use `belongs_to :awardable, polymorphic: true` and `has_many :awards, as: :awardable`  

## More helpful resources:
- [Rails Guides - Association Basics](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)  
- [Rails docs - ActiveRecord::Associations::ClassMethods](http://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#module-ActiveRecord::Associations::ClassMethods-label-Polymorphic+Associations)  
- [Railscasts Episode 154 - Polymorphic Association (Revised)](http://railscasts.com/episodes/154-polymorphic-association-revised)  
- [6ft Dan, "Rails Polymorphic Models"](http://6ftdan.com/allyourdev/2015/02/10/rails-polymorphic-models/)
- [Launch Academy - Polymorphic Associations](https://www.launchacademy.com/codecabulary/learn-rails/polymorphic-associations)
