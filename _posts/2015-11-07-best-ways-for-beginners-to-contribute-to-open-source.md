---
layout: post
title: Best Ways for Beginners To Contribute to Open Source
---

When I first started out learning to code, the idea of contributing to an open source project was really intimidating. I got that advice from everyone - "Contribute to open source! It's so easy! Employers love it!" - but I was still hesistant. I'd only been writing Ruby for a couple months; how was I going to contribute anything useful to someone else's project?

Well, I'm happy to report from the other side, it's actually pretty painless and fun for beginners at any level to make meaningful contributions to open source projects.

So shake off that [imposter syndrome](http://www.codenewbie.org/podcast/impostor-syndrome); the open source community needs you. They need everyone, in fact. Open source projects depend on a large and diverse community of contributors to really robustify their codebase. That includes people from all skill levels and all areas of expertise. Plus, the benefits are reciprocal. You'll get to know the community better, build your support system, and get your name (and your code) out there in a really positive way.

## Getting started

So where do you smart? The two best pieces of advice I got in this department were to 1) **start small** and 2) (in the immortal words of [Jerri Blank](https://www.youtube.com/watch?v=0n3DP7ADqgI)) **go with what you know.**

### Start small

Don't get caught up thinking that the only contributions open source project owners are interested in are major bugfixes or complicated refactors; that's simply not the case. Instead, it's better to zero in on a small, focused improvement.

Take Flatiron School CTO Avi Flombaum's [first open source commit](https://github.com/technoweenie/permalink_fu/commit/106c900f690d352ff2105b88c9d12bf5fb9bc9d2), for example. Here, he substituted one method in a single module (and added tests, another solid contributing move, but we'll come back to that). Clear, concise, and an irrefuteable improvement to the codebase - all in just a couple lines of code.

### Work on a project you're already using

Start with something familiar. If you're learning Rails, check out the repo for a gem you've been working with, like [factory_girl](https://github.com/thoughtbot/factory_girl) or [formtastic](https://github.com/justinfrench/formtastic/), or another Rails-based project you're using, like [rubygems](http://guides.rubygems.org/contributing/). You'll be more invested in improving the project and be able to start from a baseline working knowledge of its intended functionality.

That said, you can always explore new available projects on Github. I always like checking their [Trending page](https://github.com/trending) for the current hotness, and I actually found the source for my first open source commit through their community [Explore feature](https://github.com/explore). I was leisurely searching for Ruby-based 'cat' projects (like you do), which led me to [catsay](https://github.com/audy/catsay), a brilliant riff on the [cowsay gem](https://github.com/johnnyt/cowsay). [One commit later](https://github.com/audy/catsay/commit/2049ebb3d550bd8836d726a1e4c0100b65c536dd) and I was an open source contributor. It can really be as simple as that.

## Best ways to contribute

There's tons of easy entry points to open source contributing. Let's run through some of the best options, as well as best practices for submissions.

### 1. Report a bug

You don't have to contribute new code to open source projects; bug reporting is just as valuable (something I didn't realize right away as a beginner). Project maintainers rely on users for reporting, so as you're working with open source tools, be sure to report any bugs that you surface.

If you uncover a bug, before filing a new issue, **check and see if it's already been reported.** You don't want to clog up the Issue tracker with duplicate issues.

Once you've verified you've found a valid, unreported bug, you'll want to open a new issue and include as much information as possible for the maintainers. A quality issue report should include:

- **Clear title and description of the bug**
    Keep the title concise, but provide clear info in the description below.

    _example: "Unable to bundle install bcrypt-ruby. Added bcrypt-ruby to my Gemfile, but `bundle` command is throwing error: Failed to build gem native extension."_

- **Steps to reproduce the problem**
    Start from the beginning, and provide any necessary context.

    _example:_
    ```
    1. Add 'bcrypt-ruby' to Gemfile
    2. `cd` into Rails app directory
    3. Run `bundle`
    ```

- **System/version information (your OS version, browser information, etc.)**
  _example: "OX 10.11.1 (15B42), Ruby v2.2.0"_

- **Full text of any error messages**
    _example:_
    ```bash
    example@ubuntu:~/$ bundle
    Fetching gem metadata from http://rubygems.org/.........
    Installing bcrypt-ruby (3.0.1) with native extensions
    Gem::Installer::ExtensionBuildError: ERROR: Failed to build gem native extension.
        /usr/bin/ruby2.2.0 extconf.rb
        /usr/lib/ruby/2.2.0/rubygems/custom_require.rb:36:in `require': cannot load such file -- mkmf (LoadError)
        from /usr/lib/ruby/2.2.0/rubygems/custom_require.rb:36:in `require'
        from extconf.rb:36:in `<main>'

    An error occurred while installing bcrypt-ruby (3.0.1), and Bundler cannot continue.
    Make sure that `gem install bcrypt-ruby -v '3.0.1'` succeeds before bundling.
    ```

If there's a UI/UX component to the bug, it's helpful to include a screenshot, or better yet, an animated gif screen capture using a program like [licecap](https://github.com/lepht/licecap), a great project with a terrible name.

Want more advice on writing great bug reports? Check out Textmate's contributing wiki: https://github.com/textmate/textmate/wiki/Writing-Bug-Reports

### 2. Improve existing bug report tickets

While you're out there setting the bar for quality error reporting, you'll probably notice some open tickets that aren't up to snuff. Here's another opportunity to contribute by filling in whatever information is missing.

First, **verify that the issue is still happening**, since sometimes bugfixes go in, but the corresponding ticket is accidentally left open. If you can't reproduce the error, @ mention the maintainers so they can close the ticket.

If you're able to reproduce the issue, go ahead and add any of the information mentioned above that's missing:

- steps to reproduce
- error text
- version info
- screenshots

Any info you can add will will help take the issue one step closer to being closed.

### 3. Improve docs

All developers rely on good documentation; it's the first place you're going to go when learning new tech. As you've probably noticed, though, project docs aren't always maintained in pace with the rest of the project. You'll run into info that's unclear, outdated, or non-existent. Here's a chance for you to do yourself and other developers a huge favor - fill in those gaps in documentation.

- **Add a description**
If you run across an undocumented method, chances are it's not an easter egg. Write up a clear description, ideally with usage examples, and submit for review.

- **Write an example**
Good docs provide example usage for all methods / features / etc. ([example: Rails ActiveRecord callbacks](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)). If you run across a method that could be clarified with an example (or maybe needs a better example), add one in there. This could be your express ticket to [becoming a bonafide Rails contributor](https://github.com/rails/rails/pull/22119).

- **Fix a typo**
Here's your time to shine, English majors. Remember - (almost) [no fix is too small!](https://github.com/rails/rails/pull/21728)

### 4. Add specs

Test coverage is another area that often needs improvement, especially on smaller projects. For any public open source project, you can run the codebase through a static code analysis engine like [Code Climate](https://codeclimate.com) - or for Ruby projects, use a tool like [simplecov](https://github.com/colszowka/simplecov) to hook into Ruby's [Coverage library](http://ruby-doc.org/stdlib-2.1.0/libdoc/coverage/rdoc/Coverage.html) - to quickly spot the areas that need test coverage. Then put your test writing skills to good use!

Remember, you'll need to weight coverage vs. the total speed of the test suite. Any tests you add should be super fast or too valuable to leave out if they drag down the suite speed.

### 5. Patch a bug

Here's where you can combine everything you've learned from the four options above into the ultimate open source contribution: the bug patch. Of course, don't feel pressured to solve every bug you find, but if you have the bandwidth, take a shot at sleuthing out the solution. Even if you don't find the fix, report your findings on the existing issue, or open a new one with full details on everything you tried.

When you do find the fix, keep in mind that the best patches are as non-disruptive as possible. Specifically, your fix should **introduce as little new code as possible**, and any new code should be **written in the same style as the rest of the codebase.**  Keep your scope small; your pull request should address a single, limited issue with minimal changes overall.

When you're ready to open your pull request, be sure to cover the following:

- Reference the open issue your pull request is solving. If there's no existing ticket, open one yourself (following the [guidelines above](#1-report-a-bug)).
- Provide a clear description of your changes - what issue you're solving and how.
- Update any corresponding documentation ([as described above](#3-improve-docs)).
- Write a test case for the test suite that covers your fix ([as described above](#4-add-specs)).

## Summary

Most importantly, be patient. It might take a while for your contribution to be reviewed, but don't let that stress you out. Stay positive, keep contributing, and be proud of your new open source contributor chops.

More resources:
1. [Github contributing guidelines](https://guides.github.com/activities/contributing-to-open-source/)
2. [Github guide to finding open source projects](https://help.github.com/articles/where-can-i-find-open-source-projects-to-work-on/)
3. [Learn.co contributing guidelines](https://github.com/learn-co-curriculum/hello-world-ruby/blob/master/CONTRIBUTING.md)
4. [Rails documentation guidelines](http://guides.rubyonrails.org/api_documentation_guidelines.html)
