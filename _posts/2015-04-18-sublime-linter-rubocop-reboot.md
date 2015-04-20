---
layout: post
title: Sublime Linter Rubocop Reboot
---

This week's [Ruby Weekly](http://rubyweekly.com/issues/242) had a nice post from Matt Brictson on ["Setting up Sublime Text 3 for Rails Development"](https://mattbrictson.com/sublime-text-3-recommendations?utm_source=rubyweekly&utm_medium=email#packages). It reminded me to finally install the [SublimeLinter-rubocop package](https://packagecontrol.io/packages/SublimeLinter-rubocop). This package syncs your linter up with [rubocop](https://github.com/bbatsov/rubocop), highlighting "bad code" as you type (according to the community [Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide)). Great addition, right? 

Well, word to the wise: don't install a new linter package a couple hours before a technical interview. I didn't have a chance to tweak the default settings, which as you'll see below, are a bit...aggressive.

![Mark Style Outline]({{ site.baseurl }}/assets/mark-style-outline.png "Mark Style Outline")

I knew I'd be pair-programming with my interviewer, and no way was I gonna subject her to literally ugly code. I had to scramble to find a fix, and after jumping through a couple different sections in the [SublimeLinter documentation](http://www.sublimelinter.com/en/latest/index.html), I found the solution. Here's the lowdown. 

Turns out you have 5 different linter [mark styles](http://www.sublimelinter.com/en/latest/mark_styles.html#code-mark-styles) to choose from. The package defaults to `"mark_style": "outline"`. That style's a little visually overwhelming with my current color scheme, so I decided to switch to the `"mark_style": "none"`, so marks appear in the gutter only.

The easiest way to do this is through the [Command Palette](http://docs.sublimetext.info/en/sublime-text-3/extensibility/command_palette.html):  

1. Press `command(⌘)-shift-p` to pull up the Command Palette  
2. Type `mark` and select `SublimeLinter: Choose Mark Style`  
3. Type or click to select the mark style you want to use  
  - `fill`  
  - `outline`  
  - `solid underline`  
  - `squiggly underline`  
  - `stippled underline`  
  - `none`  

There's also a few other ways to do this:  

**Menu > Tools**  
Go to the Sublime Text menu bar and select `Tools` > `SublimeLinter` > `Mark Style`.  

**Context Menu**  
Right click within the file view to pull up the context menu, then select `SublimeLinter` > `Mark Style`  

**SublimeLinter.sublime-settings**  
Open up the `SublimeLinter.sublime-settings` file and edit `"mark_style"`:  

```json
{
    "user": {
        "debug": false,
        "error_color": "D02000",
        "lint_mode": "background",
        "linters": {
            "rubocop": {
                "@disable": false,
                "args": [],
                "excludes": []
            },
        },
        "mark_style": "none",
        "no_column_highlights_line": true,
        "passive_warnings": false,
    }
}
```

Now you've got total control over the aggressiveness of your linter highlighting. Thanks Rubocop!

![Rubocop puppies]({{ site.baseurl }}/assets/rubocop.gif "Rubocop puppies gif")
