---
layout: post
title: Debugging edeliver
---





- I've been blocked from deploying for over a week
- edeliver made things hard to debug, because edeliver error messages aren't super informative
- Elixir made things easier to debug because dependencies are available locally, so easy to add `-vvvvv` to deliver file and get more debugging output


```
GREP_COLOR="4;38"
```

This seemingly-benign line of code from my `.bash_profile`



It's been a minute since I had a really hairy bug

I was recently stumped by an issue using [edeliver](https://github.com/edeliver/edeliver) to deploy a production Elixir app.




Issue has been resolved by removing the following line from my `.bash_profile`:

```
GREP_COLOR="4;38"
```

This seemingly-benign setting was actually the cause of the invalid filename. Turns out this setting adds terminal escape sequences (that "random" ANSI code) to grep output (more info [here|https://unix.stackexchange.com/questions/350352/grep-color-auto-breaks-when-m-is-inside-colored-match], [here|https://unix.stackexchange.com/questions/116243/what-does-a-bash-sequence-033999d-mean-and-where-is-it-explained], and [here|https://unix.stackexchange.com/questions/45190/grep-color-adds-ansi-code-esck-this-can-change-displayed-text]).

Normally this wouldn't affect anything... except the edeliver script uses the `grep` command to build the filename: https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L662

Lesson learned: don't over-customize your bash.
