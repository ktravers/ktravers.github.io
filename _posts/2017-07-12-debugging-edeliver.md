---
layout: post
title: How My Bash Profile Broke edeliver
---

Yes, you read that right. A setting in my `.bash_profile` broke [edeliver](https://github.com/edeliver/edeliver), preventing me from _doing my job_ for about a week. You see, [edeliver](https://github.com/edeliver/edeliver) is the tool my team uses to deploy our Elixir apps, and it's kind of hard to ship code if you can't deploy to your staging or production environments.

Now, anyone who's [tinkered with their `.bash_profile`](http://blog.kate-travers.com/refash-your-bash/) config knows there's an infinite number of ways to totally bork your system. But this bug was well camouflaged, hiding inside a common, seemingly-benign bash setting I'd had in place for ~2.5 years without issue - a bash setting you, too, might have on your machine RIGHT NOW.

But don't worry - I tracked down the little bugger, so read on to save yourself the same hassle I went through. And for those of you in a rush, here's the [tl;dr](#solution).


## The Problem

I'd just started working on one of our Elixir projects, doing my best to learn a new codebase and a new language all at once. Things were going swimmingly until it came time to deploy changes. I could build a release, no problem, but I couldn't deploy; the command would fail every time with the following output:

```bash
$ mix edeliver deploy release to production --version=0.0.1+192-501f23d

DEPLOYING RELEASE OF IDE_VIEWER APP TO PRODUCTION HOSTS

-----> Authorizing hosts
-----> Authorizing release store host
-----> Authorizing deploy hosts on release store
-----> Uploading archive of release 0.0.1+192-501f23d from remote release store
Uploading release file failed
  source: /home/deployer/ide_viewer/builds/complete/ide_viewer_0.0.1+192-501f23d.release.tar.gz on deployer@build-server
  destination: /home/deployer/ide_viewer/ide_viewer_0.0.1+192-501f23d.tar.gz on deploy hosts
```









My bash profile broke edeliver, and it was really hard to debug, because:
- I was in a new codebase
- I'm not familiar with edeliver, Elixir, or Phoenix apps
- edeliver error messages are bogus







## Debugging Steps


## Solution

Remove any `ls` or `grep` color settings from your `.bash_profile`:

```bash
# ☠️☠️☠️ incompatible with edeliver ☠️☠️☠️
export LSCOLORS=gxxxxxxxcxxxxxcxcxgxgx
export GREP_OPTIONS='--color=always'
export GREP_COLOR='4;38'
```

These settings add [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code) to output from `ls` and `grep` commands, which are used for color formatting. Normally, the terminal interprets these escape sequences as formatting commands, not characters.











Setting colors will add terminal escape sequences to the output, which are normally escaped, and the terminal uses to color stuff.



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

This seemingly-benign setting was actually the cause of the invalid filename. Turns out this setting adds terminal escape sequences (that "random" ANSI code) to grep output (more info [here](https://unix.stackexchange.com/questions/350352/grep-color-auto-breaks-when-m-is-inside-colored-match), [here](https://unix.stackexchange.com/questions/116243/what-does-a-bash-sequence-033999d-mean-and-where-is-it-explained), and [here](https://unix.stackexchange.com/questions/45190/grep-color-adds-ansi-code-esck-this-can-change-displayed-text).

Normally this wouldn't affect anything... except the edeliver script uses the `grep` command to build the filename: https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L662
