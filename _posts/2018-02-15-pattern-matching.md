---
layout: post
title: Pattern Matching in Elixir
---

## Title ideas

Pattern Matching in Elixir

Better Elixir through Pattern Matching

Writing Confident Elixir with Pattern Matching

Declarative Elixir through Pattern Matching

Match game

Match point

PatternMatch.com

Match Maker

Perfect Match

Pattern Matchbox20

Matchless

Holding Patterns


## Outline

Outline ideas:  
- ['What is it?', 'Why use it?', 'How to Use it']


Examples in other languages (places you might already be familiar with it)  
- Regex
- Destructuring/decomposition with ES6
- JS Switch statements (more sophisticated w/ Elixir pattern matching)


Misconceptions
- Not a Design Patterns thing
- More an implementation level topic


## Basics

### Structure

pattern | match operator | value

### Evaluation

Values are determined by evaluating a match

Possible to evaluate a match without assigning any values


### Rebinding

`=` is for binding, not assignment

Can rebind by default

pin operator => don't rebind

underscore => match, but won't bind


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

## Comparisons

- JS Switch vs Elixir Case (more sophisticated w/ Elixir, allows 'partial' matches... but also just use functions instead w/ Elixir)
- Elixir case vs. Elixir Functions (use functions)
- Elixir vs Erlang (pin operator?)


## Resources

Blog posts:  
- https://elixir-lang.org/getting-started/pattern-matching.html
- https://elixirschool.com/en/lessons/basics/pattern-matching/
- https://blog.carbonfive.com/2017/10/19/pattern-matching-in-elixir-five-things-to-remember/
- https://robots.thoughtbot.com/tell-don-t-ask-in-elixir
- https://joyofelixir.com/6-pattern-matching/
- http://www.littlealchemist.io/2017-03-15-understading-elixir-pattern-matching/
- https://elixircasts.io/pattern-matching-in-elixir

Videos:  
- [ElixirConf 2015 - Confident Elixir by Lance Halvorsen](https://www.youtube.com/watch?v=E-3G7g0Dm7c)
- [Montreal Elixir - Pattern Matching in Elixir](https://www.youtube.com/watch?v=nEUHb7RJspQ&list=PLCFmW8UCDqfCA9kpbFirPEDoQYc9nCy_W&index=2&t=5s)
