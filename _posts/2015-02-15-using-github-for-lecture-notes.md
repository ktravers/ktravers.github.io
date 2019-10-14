---
layout: post
title: Using GitHub for Lecture Notes
tags: [github, flatiron school, git]
---

Flatiron School has a great system for sharing lecture notes: a dedicated GitHub repo. I forked the repo during week one, thinking I'd pull down the files to my local, take notes on the notes (Inception-style _meta-noting_), then push back up to my forked repo. No sweat.

But then comes the scary part for us git-n00bs - getting updates. I felt real comfortable pushing to / pulling from my own repositories. But to get the next days' lecture material, I was gonna have to add my instructor's repo as a remote. Yikes. Now that's a scary connection to make. Any mistakes, and I'm screwing up the lecture docs for Avi, who might have to redo them, and my classmates, who need this material asap for labs + projects. I needed to handle this delicate operation with extreme care.

![Indiana Jones Raiders of the Lost Ark animated gif]({{ site.baseurl }}/images/posts/raiders.gif "Indiana Jones Raiders of the Lost Ark animated gif")

As I now do in all these types of situations, I went straight to GitHub help docs. Here's the solution, specific to our Flatiron repo.

1) Open Terminal and `cd` to whatever folder you want to use for lecture notes storage.

2) Check the current remote repo(s) for your fork using `git remote -v`. You should probably see something like the below.

```bash
$ git remote -v
origin  git@github.com:ktravers/ruby-007-lectures-and-videos.git (fetch)
origin  git@github.com:ktravers/ruby-007-lectures-and-videos.git (push)
```

3) Time to make add Avi's source repo as a new remote _upstream_ repository. We do this using the command `git remote add [name you choose for upstream repo] [SSH clone url for upstream repo]`. You can call this remote whatever you want, but "upstream" is the usual convention.

```bash
$ git remote add upstream git@github.com:flatiron-school-ironboard/ruby-007-lectures-and-videos.git
```

4) That's it! Now just verify the new upstream repo is there by keying in `git remote -v` again. You should now see something like this:

```bash
$ git remote -v
origin  git@github.com:ktravers/ruby-007-lectures-and-videos.git (fetch)
origin  git@github.com:ktravers/ruby-007-lectures-and-videos.git (push)
upstream  git@github.com:flatiron-school-ironboard/ruby-007-lectures-and-videos.git (fetch)
upstream  git@github.com:flatiron-school-ironboard/ruby-007-lectures-and-videos.git (push)
```

Ok, we've got our upstream repo configured. Now to get the notes.

1) First be sure to add and commit any local changes using `git add .` then `git commit -m "message"`. If you push your commit, BE SURE TO PUSH TO ORIGIN using `git push origin [branch]`. **DO NOT PUSH TO UPSTREAM.** If you push to upstream, it's all over, pal.

![Corgi Death Star gif]({{ site.baseurl }}/images/posts/corgi.gif "Corgi Death Star gif")

2) Pull down new lecture material using `git pull upstream [branch]`. You'll see something like the dialogue below, noting updates and merges. Note: you might need to manually merge any changes that couldn't be auto-merged.

```bash
$ git pull upstream master
remote: Counting objects: 31, done.
remote: Compressing objects: 100% (25/25), done.
remote: Total 31 (delta 1), reused 30 (delta 0), pack-reused 0
Unpacking objects: 100% (31/31), done.
From github.com:flatiron-school-ironboard/ruby-007-lectures-and-videos
 * branch            master     -> FETCH_HEAD
   866f9f5..ccea185  master     -> upstream/master
Merge made by the 'recursive' strategy.
 rails-lecture-2/blogappwithforms/Gemfile                                         |  42 +++++++++++++
 rails-lecture-2/blogappwithforms/README.rdoc                                     |  28 +++++++++
 rails-lecture-2/blogappwithforms/Rakefile                                        |   6 ++
```

That's it! Now you've got an easy-to-maintain local directory for all your lecture materials. Easy to `subl .` into for review whenever needed. Victory dance.

![Brad Pitt dance]({{ site.baseurl }}/images/posts/brad-pitt-burn-after-reading.gif "Brad Pitt dance")

TLDR:   
- Add remote notes repo: `git remote add upstream [SSH clone url for upstream repo]`  
- Pull down new notes: `git pull upstream master`  
