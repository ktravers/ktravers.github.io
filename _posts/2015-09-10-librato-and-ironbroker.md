---
layout: post
title: Code Talk Notes - Librato and Ironbroker
---

### Librato Update

Librato has two stats types: `counter` vs `gauge`

When using counter (aka `#send_increment`), be sure to set summary function to `sum` and check `Service-Side Aggregation`.

![Librato counter]({{ site.baseurl }}/images/posts/librato-counter.png "Librato counter")

### Ironbroker Refactor

Payload == immutable  
State == mutable

Validation for each source

RabbitMq handler - rescue channel failure by creating new channel

```ruby
begin
  # do bunny stuff
  # if channel closes, rescue
rescue
  @channel = @bunny.create_channel
  # bunny keeps hopping
end
```

[Oj](https://github.com/ohler55/oj): much more efficient JSON parser

Checkout `Source` model for some cool meta-programming

### New Hierarchy Builder

Structure determined by `depth` level

Query uses Postgres's [`WITH RECURSIVE`](http://www.postgresql.org/docs/8.4/static/queries-with.html) method  
- give it an initial SELECT statement which is run once  
- give it another SELECT statment that it calls recursively until it runs out of results  
- then combines the SELECT statement  

Holds a `path` array while building, so you can see how you got to your final result

Test failures were a nightmare, but solved by creating a `rollout_helper` support method.
