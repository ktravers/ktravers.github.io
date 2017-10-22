---
layout: post
title: Get Fuzzy with LEVENSHTEIN
---

Discovered a super handy Postgres extension tonight: [`fuzzystrmatch`](https://www.postgresql.org/docs/9.4/static/fuzzystrmatch.html). This lil cutie is a real godsend when dealing with potentially crummy user input, such as, oh say, for a Rails project where you're requiring your less-than-tech-savvy relatives and future inlaws to input their email address in order to access your wedding website. Note: said website is badass, built-from-scratch, and [open source](https://github.com/ktravers/beatrix-kiddo).

Here's a visual:

![Login Modal]({{ site.baseurl }}/images/posts/wedding-modal.gif "Login Modal")

Imagining that scenario, one might be concerned about poor conversion due to typos, misspellings, or any other of the myriad problems that plague "uncontrolled" user input. And extra sadly, poor conversion here translates to a sparsely attended wedding and future full of angry inlaws... no bueno.

Enter [`fuzzystrmatch`](https://www.postgresql.org/docs/9.4/static/fuzzystrmatch.html). Adding this extension to your Postgres database gives you some super handy fuzzy string matching functions, my favorite of which so far is `levenshtein`. `levenshtein` calculates the difference between two strings, aka the "edit distance", and it's based on [this metric](https://en.wikipedia.org/wiki/Levenshtein_distance) named after [this guy](https://en.wikipedia.org/wiki/Vladimir_Levenshtein).

You can use it to make some extra forgiving ActiveRecord queries, as laid out below.

```
> User.find_by('levenshtein(lower(email), lower(?)) <= 3', 'test@email.c')
=> #<User id: 1, email: "test@email.com">

> User.find_by('levenshtein(lower(email), lower(?)) <= 3', 'TEST@EMAILCOM')
=> #<User id: 1, email: "test@email.com">

> User.find_by('levenshtein(lower(email), lower(?)) <= 3', ' test@email.com ')
=> #<User id: 1, email: "test@email.com">

> User.find_by('levenshtein(lower(email), lower(?)) <= 3', 'test@@email.com')
=> #<User id: 1, email: "test@email.com">

> User.find_by('levenshtein(lower(email), lower(?)) <= 3', 'teste@mail.com')
=> #<User id: 1, email: "test@email.com">
```

I'm using it to make it extra easy for folks to log in, provided they can at least remember their own email within a 3 character margin of error. Here's hoping.

## Super Basic Implementation:

### Add migrations

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :email, null: false, default: ""
    end
  end
end

class EnableFuzzystrmatchExtension < ActiveRecord::Migration
  def change
    enable_extension 'fuzzystrmatch'
  end
end
```

### Add class method on model

```ruby
class User < ActiveRecord::Base
  def self.fuzzy_match(email)
    find_by('levenshtein(lower(email), lower(?)) <= 3', email.strip)
  end
end
```

### Query from controller action

```ruby
class SessionsController < ApplicationController

  def create
    user = User.fuzzy_match(user_params[:email])

    if user
      # login user
    else
      # redirect with flash error
    end
  end

  private

  def user_params
    params.require(:user).permit(:email)
  end
end
```
