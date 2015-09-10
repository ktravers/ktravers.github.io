---
layout: post
title: Code Talk Notes - Librato and Ironbroker
---

### Librato Update

Librato has two stats types: `counter` vs `gauge`

When using counter (aka `#send_increment`), be sure to set summary function to `sum` and check `Service-Side Aggregation`.

![Librato counter]({{ site.baseurl }}/assets/librato-counter.png "Librato counter")

### Ironbroker Refactor

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
