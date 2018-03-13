---
layout: post
title: Pattern Matching in Elixir
---

Think back to the last time someone tried to sell you on some shiny, new tech. Maybe it was (yet another) a Javascript framework. Maybe it was non-relational databases. Or maybe it was this cool new functional programming language everyone's talking about: [Elixir](https://elixir-lang.org/).

As programmers, we hear these pitches all the time, so it takes something truly compelling to win us over.

For me with Elixir, that thing was [pattern matching](https://elixir-lang.org/getting-started/pattern-matching.html). It's a core paradigm of the language and a great entry point for exploring all the things that make Elixir such an awesome, powerful, and fun language to develop in.

So let's break down what it is and why it's so great.

### The Match Operator

To understand pattern matching in Elixir, start by reframing the way you think about tying values to variables. Take the statement `x = 1`. You probably read that as "x equals 1", where we're assigning the value `1` to the variable `x`, right?

Welp, not in Elixir.

In that statement, the `=` is known as the "match operator", and it's not doing any assigning. Instead, it's evaluating whether the value on the right _matches_ the pattern on the left. If it's a match, then the value is bound to the variable [[1](#footnotes)]. If not, then a `MatchError` is raised.

| x     | =            | 1   |
|:-----:|:------------:|:---:|
|pattern|match operator|value|

What does it mean to "match"? It means the value on the right matches the form and sequence of the pattern on the left.


### Simple Examples

Let's run through the basics of pattern matching with these simple examples below.

#### Binding on Match

```elixir
x = 1
```

Here, the match evaluates to true, since anything on the right-hand-side will match on an empty variable, so the empty variable on the left is bound to the value on the right.

#### Match Without Binding

```elixir
x = 1
1 = x
```

Both of these statements are valid expressions, and they also both match (!!!)

In the top expression, the match evaluates to true and the value is bound to the variable. In the bottom expression, the match evaluates to true, but nothing is bound, since variables can only be bound on the left side of the `=` match operator. For example, the statement `2 = y` would throw a `CompileError`, since `y` is not defined.

#### Re-binding

```elixir
x = 1
x = 2
```

If you pattern match on a bound variable, like `x` above, it will be rebound if it matches.

#### Pin Operator

```elixir
 x = 1
^x = 2
#=> ** (MatchError) no match of right hand side value: 2
```

If you don't want the variable to be rebound on match, use the `^` [pin operator](https://elixir-lang.org/getting-started/pattern-matching.html#the-pin-operator). The pin operator prevents the variable from being rebound by forcing a strict match against its existing value.

#### Lists

```elixir
iex(1)> [a, b, c] = [1, 2, 3]
iex(2)> a
#=> 1
iex(3)> b
#=> 2
iex(4)> c
#=> 3
```

We can pattern match on more complex data structures, like lists. Again, any left side variables will bound on a match.

#### List `[head | tail]` Format

```elixir
iex(1)> [head | tail] = [1,2,3,4]
iex(2)> head
#=> 1
iex(3)> tail
#=> [2,3,4]
```

One cool thing you can do with lists is pattern match on the head and tail. Use the `|` syntax to bind the leftmost variable to the first element in the list and the remaining elements to the rightmost variable (these variables don't have to be named `head` and `tail`; you can choose any names you want).

This syntax comes in handy when you have a list of elements you want to operate on one-by-one, since it allows you to recursively iterate over the list very cleanly and succinctly.

```elixir
iex(1)> list = [2,3,4]
iex(2)> [1 | list]
#=> [1,2,3,4]
```

You can use this syntax to prepend elements to lists, too, if you're feeling fancy.

```elixir
iex(1)> [first | rest] = []
#=> ** (MatchError) no match of right hand side value: []
```

Watch out for empty lists, though. You'll raise a `MatchError` if you use this syntax on an empty list, since there's nothing to bind either variable to.

#### Match Errors

```elixir
iex(1)> [x,y] = [4,5,6,7]
#=> ** (MatchError) no match of right hand side value: [4,5,6,7]
```

Keep in mind, the match will fail if you compare different size lists.

```elixir
iex(1)> [foo, bar] = {:foo, :bar}
#=> ** (MatchError) no match of right hand side value: {:foo, :bar}
```

Matches also fail if you try to compare two different data structures, like a list and a tuple.

#### Tuples

```elixir
iex(1)> {a, b, c} = {1,2,3}
iex(2)> a
#=> 1
iex(3)> b
#=> 2
iex(4)> c
#=> 3
```

Pattern matching with tuples operates much the same as with lists.

```elixir
iex(1)> {:ok, message} = {:ok, "success"}
iex(2)> message
#=> "success"
iex(3)> {:ok, message} = {:error, "womp womp"}
#=> ** (MatchError) no match of right hand side value: {:error, "womp womp"}
```

One common pattern you'll see in Elixir is functions returning tuples where the first element is an atom that signals status, like `:ok` or `:error`, and the second element is a string message.

#### `_` Underscore Variable

```elixir
iex(1)> {_, message} = {:ok, "success"}
iex(2)> message
#=> "success"
iex(3)> {_, message} = {:error, "bummer"}
iex(4)> message
#=> "bummer"
iex(5)> [ head | _ ] = [1,2,3,4]
iex(6)> head
#=> 1
```

For times when you want to pattern match but don't care about capturing any values, you can use the `_` underscore variable. This special reserved variable matches everything; it's a perfect catch-all.

```elixir
iex(1)> {_, message} = {:ok, "success"}
iex(2)> _
#=> ** (CompileError) iex:2: unbound variable _
```

Just be aware that `_` really truly is a throw-away variable, in that you can't read from it. If you try, Elixir will throw a `CompileError`.

### So what's the big deal?

Maybe you're not blown away by the examples above. Elixir has some nice syntactic sugar for pattern matching... but what's so groundbreaking about that?

![Shania Twain That Don't Impress Me Much]({{ site.baseurl }}/images/posts/shania-not-impressed.jpg "Shania Twain That Don't Impress Me Much")

Let's take a look at some practical real world applications.

### Real World Examples

We'll start with a problem that's probably familiar to most web developers: displaying public-facing user “display names” based on user-inputted data.

This was something I worked on recently in the [Learn.co](https://learn.co) codebase. On our site, we like to encourage an active, friendly sense of community, so we display user's names (built from information volunteered by the user) in lots of places across the site, including the [Ask a Question chat feature](http://help.learn.co/ask-a-question/where-can-i-ask-a-question-about-a-lesson).

![Learn.co Ask a Question]({{ site.baseurl }}/images/posts/learn-co-aaq.png "Learn.co Ask a Question")

Problem is, we don't require users to give us their full name or even set a username, so when it comes to building a public-facing display name, there's no guarantee that any identifying information - first name, last name, or username - is available. Additionally, all of this information is inputted manually by the user, and while we sanitize it to some degree before persisting, weird stuff can still get through.

To address this problem, our product team developed the following requirements:  
  1. If the user has provided their first and last name, display both together as their full name
  2. If we don't have first or last name, check if the user has provided their username, and if yes, display the username in place of full name
  3. If we don't have any of the above, display a reasonable generic default (here, we'll just use "New User")

How could we represent these conditions in code?

#### Javascript Example

Writing that function in Javascript might look something like this:&#42;

```javascript
export const displayName = (user) => {
  if (user.firstName.length > 0) {
    if (user.lastName.length > 0) {
      return `${user.firstName} ${user.lastName}`.trim();
    } else {
      return `${user.firstName}`.trim();
    }
  } else if (user.username.length > 0) {
    return user.username;
  } else {
    return 'New User';
  }
}
```

&#42; I realize these examples are somewhat contrived, but bear with me. They're for illustrative purposes, not code review.

There's a lot of things that make this function pretty hard to grok in one glance. First off, there's Javascript's punctuation-heavy syntax, which can be a little rough on the eyes if you've been away from it for a little while. All the nested conditionals add complexity, too, as well as mental overload. Then additionally, we're also doing some nil checking (via `length`) and throwing in some string sanitation for good measure. All-in-all, not super readable.

#### Ruby Example

If we switch to Ruby, a language praised for being "developer-friendly", the situation doesn't improve much.

```ruby
def display_name(user)
  if user.first_name.length > 0
    if user.last_name.length > 0
      "#{user.first_name} #{user.last_name}".strip
    else
      "#{user.first_name}".strip
    end
  elsif user.username.length > 0
    user.username
  else
    'New User'
  end
end
```

We still have our nested conditionals, and this long, "pointy" method decidedly does not pass the ["squint test"](https://atom.io/packages/squint-test).

#### Elixir Example

Let's see if we can fare any better with Elixir.


```elixir
defmodule Account do
  def display_name(%{first: first, last: last}) do
    String.trim("#{first} #{last}")
  end

  def display_name(%{username: username}), do: "#{username}"

  def display_name(_), do: “New User”
end
```

Here, each conditional has been separated out into its own function clause. Unlike in other languages like Ruby, when we "overload" a function like this (e.g. make multiple function declarations with the same function name), we're not overwriting the original function. Instead, these are known as multi-clause functions, and when you call a function that has several clauses, it'll will try each clause (starting at the top of the file and moving down) until it finds one that matches.

You want to put your most specific clauses at the top, since those will match first. If you put something too general at the top, then it'll match everything and none of the clauses below it will ever get hit. Luckily, Elixir is pretty cool and usually throws a warning if you make this mistake.

Multi-clause functions allow us to break our conditional logic into the smallest, atomic pieces, thereby keeping it isolated, encapsulated, and much more legible. It's easy to tell at a glance what each of these function clauses is doing.

#### Handling the Un-Happy Path

But you might have noticed our Elixir example here has a bit of an unfair advantage. Most of the added complexity in the Ruby and Javascript examples came from handling `nil` cases, and we're not checking for those at all in the Elixir example - yet.

You might be tempted to throw a `case` statement into the first `display_name/1` function clause. You'll want to resist, though, because `case` statements are not The Elixir Way™. Before reaching for `case`, try adding more clauses with higher specificity instead:

```elixir
defmodule Account do
  # Nil checks
  def display_name(%{first: nil, last: nil, username: nil}), do: display_name(%{})
  def display_name(%{first: nil, last: nil, username: username}) do
    display_name(%{username: username})
  end
  def display_name(%{first: nil, last: nil}), do: display_name(%{})

  # Happy paths
  def display_name(%{first: first, last: last}), do: do_trim("#{first} #{last}")
  def display_name(%{username: username}), do: "#{username}"
  def display_name(_), do: “New User”
end
```

Remember, clause order matters. We'll want to add our nil check clauses to the top of the module, so they'll match first. We can leverage some recursive goodness here, too, by calling `display_name/1` from inside `display_name/1`, passing it arguments we know it'll match on as expected.

#### Guard Clauses

Let's throw one more curveball into the mix. There's another case we need to guard against with display names, and that's where a user has given us their full name, but it contains [personal identifying information (PII)](https://en.wikipedia.org/wiki/Personally_identifiable_information).

This situation actually happens not infrequently on Learn.co. For some reason on our public, free [Bootcamp Prep course sign up page](https://learn.co/sign_up), users often enter their email in the full name field.

![Learn.co Bootcamp Prep course sign up form]({{ site.baseurl }}/images/posts/learn-co-sign-up-annotated.png "Learn.co Bootcamp Prep course sign up form")

Clearly, we need to change something about this UI. But in the meantime, we can protect against displaying PII via some more complex pattern matching techniques.

So far, our `display_name/1` function clauses look like this:

```
defmodule Account do
  # Nil checks
  def display_name(%{first: nil, last: nil, username: nil}), do: display_name(%{})
  def display_name(%{first: nil, last: nil, username: username}) do
    display_name(%{username: username})
  end
  def display_name(%{first: nil, last: nil}), do: display_name(%{})

  # Happy paths
  def display_name(%{first: first, last: last}), do: do_trim("#{first} #{last}")
  def display_name(%{username: username}), do: "#{username}"
  def display_name(_), do: “New User”
end
```

Let's expand our arsenal a bit by adding two more function clauses that guard against PII in a user's first name:

```elixir
defmodule Account do
  #...(Nil checks)

  def display_name(%{first: first, username: nil}) when first =~ "@" do
    "<<Redacted>>"
  end

  def display_name(%{first: first, username: username}) when first =~ "@" do
    "#{username}"
  end

  #... (Happy paths)
end
```

Elixir function declarations support [guard clauses](https://hexdocs.pm/elixir/master/guards.html), which is what the `when first =~ "@"` is in the examples above. Guard clauses are a nice way to match against more complex patterns without adding too much clutter to your functions. Only a handful of expressions are supported, and they're meant to be short and sweet.

#### Wrap Up

Thanks to the power of pattern matching and multi-clause functions, we now have clear, clean, and effective code to handle displaying user names. And as new requirements come up, we don't have to touch any of these existing methods. We can simply add new clauses as needed.

```elixir
defmodule Account do
  def display_name(%{first: nil, last: nil, username: nil})
  def display_name(%{first: nil, last: nil, username: username})
  def display_name(%{first: first, username: nil}) when first =~ "@"
  def display_name(%{first: first, username: username}) when first =~ "@"
  def display_name(%{first: first, last: last})
  def display_name(%{username: username})
  def display_name(_)
end
```

### Takeaways

As mentioned back at the beginning, working with pattern matching in Elixir requires you to think a bit differently - but different in a good way. The way the language is designed - the paradigms it embraces, the functionality it supports - encourages you to follow general programming best practices. Pattern matching is one of the best examples of this.

Take pattern matching on multi-clause functions. By supporting this, Elixir nudges you towards writing small, declarative functions - short functions that do one thing only, e.g. functions that follow the [Single Responsibility Principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

Likewise, by declaring the pattern you want to match against, you're sending a clear signal about what inputs you expect to receive. Your code becomes more self-documenting by default.

All-in-all, Elixir encourages you to write code that’s 1) declarative 2) self-documenting and 3) well scoped. It’s helping you become a stronger programmer.

Now that's impressive.

![That Does Impress Me Much]({{ site.baseurl }}/images/posts/impress.gif "That Does Impress Me Much")

Any questions? Leave them in the comments below. Thanks for reading!

### Resources

#### Readings:
- Elixir docs: [Pattern Matching](https://elixir-lang.org/getting-started/pattern-matching.html)
- Elixir School: [Pattern Matching](https://elixirschool.com/en/lessons/basics/functions/#pattern-matching)
- Anna Neyzberg, ["Pattern Matching in Elixir: Five Things to Remember"](https://blog.carbonfive.com/2017/10/19/pattern-matching-in-elixir-five-things-to-remember/)

#### Videos:
- Joao Goncalves, ["Getting Started with Elixir: Pattern Matching versus Assignment"](https://www.youtube.com/watch?v=zwPqQngLn9w&index=6&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W)
- Dave Thomas, [Think Different (ElixirConf2014 Keynote)](https://www.youtube.com/watch?v=5hDVftaPQwY)
- Lance Halvorsen, ["Confident Elixir" (ElixirConf 2015)](https://www.youtube.com/watch?v=E-3G7g0Dm7c)

#### Tutorials:
- Code School, [Try Elixir - Pattern Matching](http://campus.codeschool.com/courses/try-elixir/level/3/section/1/pattern-matching)

### Footnotes

[1] Binding vs. Assignment

The distinction between variable binding vs. variable assignment is small, but critical when it comes to pattern matching in Elixir. For any readers familiar with Erlang, all the binding and re-binding variables above may have seemed odd. In Erlang, variables are immutable, and since Elixir is built on top of the Erlang VM, variables are immutable in Elixir, too.

If variables are immutable, then why are we allowed to tie and re-tie values to variables with pattern matching?

We have to drop down to machine-level memory management to get the answer. Assignment assigns data to a place in memory, so re-assigning a variable changes the data in place. Binding creates a reference to a place in memory, so re-binding just changes the reference, not the data itself.

Think of the variable as a suitcase. Binding the variable is like slapping a label on the suitcase. Assigning is like swapping out the contents [[source](https://stackoverflow.com/a/48103225/3880374)].

For more context, Elixir creator José Valim has a nice post on [Comparing Elixir and Erlang variables](http://blog.plataformatec.com.br/2016/01/comparing-elixir-and-erlang-variables/).
