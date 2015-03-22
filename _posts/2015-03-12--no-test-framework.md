---
layout: post
title: --no-test-framework
---
### Note to self:

[ ![Bart Simpson chalkboard]({{ site.baseurl }}/assets/bart-simpson-generator.gif "Bart Simpson chalkboard generator") ](http://www.addletters.com/pictures/bart-simpson-generator/5302585.htm#.VQJJlxDF-5I "Chalkboard Generator")

Why? Without the `--no-test-framework` flag, Rails `generate` will do the following:   

a) overwrite any existing tests that you, your colleagues, or your instructors have already written for that particular object    

b) generate unnecessary / unwanted tests (see [Thoughtbot's helpful post on diminishing test coverage returns](https://robots.thoughtbot.com/unit-and-functional-tests-are-as-useful-as-100-code)).   

Better to write your own tests. Here's hoping repetition leads to retention...