---
layout: post
title: JS Testing with Jest and React
tags: [code reading, javascript, testing]
---

A code reading by [@gj](https://github.com/gj)

## [Jest](https://facebook.github.io/jest/)

- New hotness for testing React
- More or less a superset of [Jasmine](https://jasmine.github.io/)
- Backed by Facebook
- "Just works" out of the box
- Started as a wrapper around [JSDom](https://github.com/jsdom/jsdom), evolved into its own thing

Hopes and dreams: [Jest](https://facebook.github.io/jest/) makes test writing fun and easy => we write more tests

### Helpful features

1. Watch mode (`--watch`) - runs test on code changes (similar to `guard` on server-side)
2. Coverage metrics (`--coverage`) - out of the box
3. Notifications mode (`--notify`) - native OS notification when test runs completes
4. Tracks test failures - runs failed tests first on next run (shorter feedback loop)
5. Tracks test suite run time - suites run in parallel across processes
6. Fully sandboxed - tests are strictly scoped, so no globals pollution

### Suggestions

1. Enzyme's `shallow` over `mount` for rendering components

WIP... more to follow after next week's code reading.
