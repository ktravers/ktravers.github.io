---
layout: post
title: Pattern Matching in Elixir
---

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


## Resources

Readings:  
- https://elixir-lang.org/getting-started/pattern-matching.html
- https://elixirschool.com/en/lessons/basics/pattern-matching/
https://elixirschool.com/en/lessons/basics/functions/#pattern-matching
- https://blog.carbonfive.com/2017/10/19/pattern-matching-in-elixir-five-things-to-remember/
- https://joyofelixir.com/6-pattern-matching/
- http://www.littlealchemist.io/2017-03-15-understading-elixir-pattern-matching/

Videos:  
- [Getting Started with Elixir : Pattern Matching versus Assignment by Joao Goncalves](https://www.youtube.com/watch?v=zwPqQngLn9w&index=6&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W)
- [Keynote: Think Different by Dave Thomas](https://www.youtube.com/watch?v=5hDVftaPQwY)
- [ElixirConf 2015 - Confident Elixir by Lance Halvorsen](https://www.youtube.com/watch?v=E-3G7g0Dm7c)
- [Pattern Matching in Elixir (ElixirCasts)](https://elixircasts.io/pattern-matching-in-elixir#)
- [Montreal Elixir - Pattern Matching in Elixir by Mark Crisp](https://www.youtube.com/watch?v=nEUHb7RJspQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=2&t=5s)
- [ElixirConf.eu Lightning talks - Pattern Matching by Wojtek Mach](https://www.youtube.com/watch?v=pOR1z8sAjzQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=3)
- [Elixir bits - recursion, guards, and pattern matching by Isaac Ben Hutta](https://www.youtube.com/watch?v=jVl-Pp1iELQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=7)


Tutorials:  
- [Code School Try Elixir - Pattern Matching](http://campus.codeschool.com/courses/try-elixir/level/3/section/1/pattern-matching)
