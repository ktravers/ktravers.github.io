---
layout: post
title: Pattern Matching in Elixir
---

Think back to the last time someone tried to sell you on some shiny, new tech. Maybe it was (yet another) a Javascript framework. Maybe it was non-relational databases. Or maybe it was this cool new functional programming language everyone's talking about: [Elixir](https://elixir-lang.org/).

As programmers, we hear these pitches all the time, so it takes something truly compelling to win us over.

For me with Elixir, that thing was pattern matching. It's a key feature of the language and a great entry point for exploring all the things that make Elixir such an awesome, powerful, and fun language to develop in.

So let's break down what it is and why it's so great.

### The Match Operator

To understand pattern matching in Elixir, you have to reframe the way you think about programming right off the bat. Take the statement `x = 1`. You probably read that as "x equals 1", where we're assigning the value `1` to the variable `x`, right?

Welp, not in Elixir.

In that statement, the `=` operator is known as the "match operator", and it's not assigning anything. Instead, it's evaluating whether the value on the right _matches_ the pattern on the left. If it's a match, then the value is bound to the variable.[1] If not, then a `MatchError` is raised.

| x     | =            | 1   |
|:-----:|:------------:|:---:|
|pattern|match operator|value|

What does it mean to "match"? It means the value on the right matches the form and sequence of the pattern on the left.


### Simple Examples

Let's run through the basics of pattern matching with these simple examples below.


```elixir
x = 1
```

Here, the match evaluates to true, since an empty variable matches anything on the right-hand-side, so the empty variable on the left is bound to the value on the right.

```elixir
x = 1
1 = x
```

Both of these statements are valid expressions, and they also both match, since the integer `1` matches the bound value of `x`.

In the top expression, the match evaluates to true and the value is bound to the variable. In the bottom expression, the match evaluates to true, but nothing is bound, since variables can only be bound on the left side of the `=` match operator. For example, the statement `2 = y` would throw a `CompileError`, since `y` is not defined.

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
iex(4) c
#=> 3
```

We can pattern match on more complex data structures, like lists. Again, any left side variables will bound on a match.

```elixir
iex(1)> [head | tail] = [1,2,3,4]
iex(2)> head
#=> 1
iex(3)> tail
#=> [2,3,4]
```

One cool thing you can do with lists is pattern match on the head and tail. Use the `|` syntax to bind the leftmost variable to the first element in the list and the remaining elements to the rightmost variable (these don't have to be `head` and `tail`; you can choose any variable names you want).

This syntax comes in handy when you have a list of elements you want to operate on one-by-one, since it allows you to recursively iterate over the list very cleanly and succinctly.

```elixir
iex(1)> list = [2,3,4]
iex(2)> [1 | list]
#=> [1,2,3,4]
```

You can use this syntax to prepend elements to lists, too, if you're feeling fancy.


```elixir
iex(4)> [first | rest] = []
#=> ** (MatchError) no match of right hand side value: []
```

Watch out for empty lists, though. You'll raise a `MatchError` if you use this syntax on an empty list, since there's nothing to bind either variable to.

```elixir
iex(1)> [x,y] = [4,5,6,7]
#=> ** (MatchError) no match of right hand side value: [4,5,6,7]
```

Keep in mind, the match will fail if you compare different size lists.

```elixir
iex(2)> [foo, bar] = {:foo, :bar}
#=> ** (MatchError) no match of right hand side value: {:foo, :bar}
```

Matches also fail if you try to compare two different data structures, like a list and a tuple.


#### Tuples






[1] Binding vs assignment


## Outline

* Intro
  * Story about first loves
  * Pattern matching was how I fell in love with Elixir
  * Cool thing you learn early, sells you on the language right away
* What is Pattern Matching?
  * Match operator
  * Binding != Assignment
  * Simple Examples
* Why Pattern Matching?
  * Declarative
  * Self documenting (legible) ("Tells a good story" - Lance)
  * Encapsulated (w/ multi-clause functions)
* Practical Examples





###### SCRATCH PAD

## Title ideas

Pattern Matching in Elixir, Better Elixir through Pattern Matching, Writing Confident Elixir with Pattern Matching, Declarative Elixir through Pattern Matching, Match game, Match point, PatternMatch.com, Match Maker, Perfect Match, Pattern Matchbox20, Matchless, Holding Patterns

Outline ideas:  
- ['What is it?', 'Why use it?', 'How to Use it']


Examples in other languages (places you might already be familiar with it)  
- Regex
- Destructuring/decomposition with ES6
- JS Switch statements (more sophisticated w/ Elixir pattern matching)
- Card games (c/o Wojtek Mach ElixirConf.eu Lightning Talk)


Misconceptions
- Not a Design Patterns thing
- More an implementation level topic


From Packt video series:  
> What is pattern matching and how does it differ from assignment? How can we use pattern matching?
>
> - Introduce the concept of assignment
> - Define pattern matching as opposed to assignment
> - Present the pin operator
>
> How can we leverage pattern matching to extract data from complex structures?
>
> - Introduce pattern matching in collection types
> - Present the binary data type in Elixir
> - Extract binary data using pattern matching
>
> How can we leverage pattern matching in function calls and how can we match a call with specific arguments to a concrete function?
>
> - Present a use case and a concrete problem
> - Provide a solution for the problem using pattern matching
> - Introduce function guard clauses and complete use case


## Intro

Why pattern matching? One of the things that hooked me on / sold me on Elixir right off the bat.

Super cool. See the value right away. One of the best things about the language. Don't have to get deep to love it and see it's utility.


## Basics

### Structure

pattern | match operator | value


### Evaluation

Values are determined by evaluating a match

Possible to evaluate a match without assigning any values


### Binding/Rebinding

`=` is for binding, not assignment

Can rebind by default

pin operator => don't rebind

underscore => match, but won't bind


```ruby
# looks the same, but doing something very different

a = 12 # ruby assignment

a = 20 # elixir binding
```



### Literals

Lists: can use `[head | rest]`

Maps, Lists, Structs


### Where to use

used in assertions, function heads, case statements

also multi clause functions (multiple clauses of a function w/ same name and arity)


## Advantages

Better than branching conditions bc  
  - shorter, more focused story
  - behavior encapsulated into separate spaces
  - "tell a good story" (self documenting code)
  - avoid error handling for control flow (no begin rescues)
  - code for the happy path

Think of the termination case, then happy path

## Comparisons

- JS Switch vs Elixir Case (more sophisticated w/ Elixir, allows 'partial' matches... but also just use functions instead w/ Elixir)
- Elixir case vs. Elixir Functions (use functions)
- Elixir vs Erlang (pin operator?)


## Examples

- Texas Holdem
- Markdown Parsing
- HTTP Responses


### Resources

#### Readings:
- https://elixir-lang.org/getting-started/pattern-matching.html
- https://elixirschool.com/en/lessons/basics/pattern-matching/
https://elixirschool.com/en/lessons/basics/functions/#pattern-matching
- https://blog.carbonfive.com/2017/10/19/pattern-matching-in-elixir-five-things-to-remember/
- https://joyofelixir.com/6-pattern-matching/
- http://www.littlealchemist.io/2017-03-15-understading-elixir-pattern-matching/

#### Videos:
- [Getting Started with Elixir : Pattern Matching versus Assignment by Joao Goncalves](https://www.youtube.com/watch?v=zwPqQngLn9w&index=6&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W)
- [Keynote: Think Different by Dave Thomas](https://www.youtube.com/watch?v=5hDVftaPQwY)
- [ElixirConf 2015 - Confident Elixir by Lance Halvorsen](https://www.youtube.com/watch?v=E-3G7g0Dm7c)
- [Pattern Matching in Elixir (ElixirCasts)](https://elixircasts.io/pattern-matching-in-elixir#)
- [Montreal Elixir - Pattern Matching in Elixir by Mark Crisp](https://www.youtube.com/watch?v=nEUHb7RJspQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=2&t=5s)
- [ElixirConf.eu Lightning talks - Pattern Matching by Wojtek Mach](https://www.youtube.com/watch?v=pOR1z8sAjzQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=3)
- [Elixir bits - recursion, guards, and pattern matching by Isaac Ben Hutta](https://www.youtube.com/watch?v=jVl-Pp1iELQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=7)


#### Tutorials:
- [Code School Try Elixir - Pattern Matching](http://campus.codeschool.com/courses/try-elixir/level/3/section/1/pattern-matching)
