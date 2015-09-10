---
layout: post
title: Code Talk Notes - Librato and Ironbroker
---

### Librato Update

Librato has two stats types: `counter` vs `gauge`

When using counter (aka `#send_increment`), be sure to set summary function to `sum` and check `Service-Side Aggregation`.

### Ironbroker Refactor

Validation for each source

