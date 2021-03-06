---
layout: post
title: How My Bash Color Settings Broke edeliver
tags: [bash, elixir, deployment]
---

Yep, you read that right. My bash color settings broke [edeliver](https://github.com/edeliver/edeliver), the tool my team uses to deploy our Elixir apps.

Now, anyone who's [tinkered with their `.bash_profile`](http://blog.kate-travers.com/refash-your-bash/) knows there's an infinite number of ways to totally bork your system. But this bug was well camouflaged, hiding inside a common, seemingly-benign bash setting I'd had in place for over two and a half years without issue - a bash setting you, too, might have on your machine RIGHT NOW 😱

But don't worry, I tracked down the little bugger, so read on to save yourself the same hassle I went through. And for those of you in a rush, here's the [tl;dr](#the-solution).

- [The Problem](#the-problem)
- [Debugging Steps](#debugging-steps)
- [The Turning Point](#the-turning-point)
- [The Explanation](#the-explanation)
- [The Solution](#the-solution)
- [Learnings](#learnings)


## The Problem

I'd just started working on one of our Elixir projects, doing my best to learn a new codebase and a new language all at once. Things were going well until it came time to deploy changes. I could [build a release](https://github.com/edeliver/edeliver#build-commands), no problem, but I couldn't [deploy](https://github.com/edeliver/edeliver#deploy-commands); the command would fail every time with the following output:

```bash
$ mix edeliver deploy release to production --version=X.X.X-XXX-XXXXXXX --verbose

DEPLOYING RELEASE OF MY_APP APP TO PRODUCTION HOSTS

-----> Authorizing hosts
-----> Authorizing release store host
-----> Authorizing deploy hosts on release store
-----> Uploading archive of release X.X.X-XXX-XXXXXXX from remote release store
Uploading release file failed
  source: /home/deployer/my_app/builds/complete/my_app_X.X.X-XXX-XXXXXXX.release.tar.gz on deployer@host
  destination: /home/deployer/my_app/my_app_X.X.X-XXX-XXXXXXX.tar.gz on deploy hosts
```

My teammates _could_ deploy using the same command; I was the only one getting this error. I tried running the command from my work and personal computers, and it errored out the same way consistently on both machines (I use shared dotfiles, so this wasn't super surprising).


## Debugging Steps

- [Hard Reset](#hard-reset)
- [Get Verbose](#get-verbose)
- [Manual Workaround](#manual-workaround)
- [SSH Sanity Check](#ssh-sanity-check)

### 1. Hard reset

My first recourse here was the same I use any time I hit a weird error: start over fresh. I deleted the project directory from my machine, re-cloned it back down, and tried deploying the master branch (which was already running on production). No dice.

**Outcome:**  
✅ Confirmed issue wasn't with something I'd changed in the codebase  
❌ Didn't work. Deploy still failed.


### 2. Get Verbose

I was running the deploy command with the `--verbose` flag, but the edeliver error message wasn't telling me much about _why_ the command was failing (more on that [later](#learnings)). I needed more info.

On my teammate Steven's excellent advice, I opened up `my_app/deps/edeliver`, found the command that was failing by searching for the error message ("Uploading release file failed"), and flipped it into verbose debug mode by adding the `-vvvvv` option (more `v`'s == more verbose):

```bash
# https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L685-L759

# copies the release archive to the production hosts to the given path.
upload_release_archive() {
  # lotsa code, omitted here for brevity
  ssh -A -vvvvv -o ConnectTimeout="$SSH_TIMEOUT" "$_release_store_host" "$_remote_job $SILENCE" || \
    error "Uploading release file failed\n  source: ${_release_store_path%%/}/${_release_file} on $_release_store_host\n  destination: $_dest_file_name on deploy hosts"
}
```

This gave me way more insight into what this script was trying (and failing) to do. Honestly, probably too much insight. Re-running the deploy command in verbose debug mode gave me back ~200 lines of output:

```bash
$ mix edeliver deploy release to production --version=X.X.X-XXX-XXXXXXX --verbose

DEPLOYING RELEASE OF MY_APP APP TO PRODUCTION HOSTS

-----> Authorizing hosts
-----> Authorizing release store host
-----> Authorizing deploy hosts on release store
-----> Uploading archive of release X.X.X-XXX-XXXXXXX from remote release store
OpenSSH_7.4p1, LibreSSL 2.5.0
debug1: Reading configuration data /Users/kate/.ssh/config
debug1: /Users/kate/.ssh/config line 1: Applying options for *
debug1: Reading configuration data /etc/ssh/ssh_config
debug2: resolving "xxx.xxx.xx.xxx" port XXX
debug2: ssh_connect_direct: needpriv 0
debug1: Connecting to xxx.xxx.xx.xxx [xxx.xxx.xx.xxx] port XXX.
debug2: fd 3 setting O_NONBLOCK
debug1: fd 3 clearing O_NONBLOCK
debug1: Connection established.
debug3: timeout: 99999969 ms remain after connect
debug1: identity file /Users/kate/.ssh/id_rsa type 1

### approx 180 more lines...

debug3: send packet: type 1
debug1: fd 0 clearing O_NONBLOCK
debug1: fd 1 clearing O_NONBLOCK
debug3: fd 2 is not O_NONBLOCK
Transferred: sent 4624, received 2996 bytes, in 0.2 seconds
Bytes per second: sent 20610.0, received 13353.7
debug1: Exit status 1
Uploading release file failed
  source: /home/deployer/my_app/builds/complete/my_app_X.X.X-XXX-XXXXXXX.release.tar.gz on deployer@host
  destination: /home/deployer/my_app/my_app_X.X.X-XXX-XXXXXXX.tar.gz on deploy hosts
```

Steven did the same thing on his machine, so we'd have his successful debug log for comparison.

His output also gave us another valuable piece of information: the actual bash command [`$_remote_job`](https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L754-L755) built by the [`upload_release_archive` function](https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L689), i.e. the one that copies the release from our build server to our production server, i.e. the one throwing the error. We'd follow that lead next.

**Outcome:**  
✅ Better output with verbose debug mode  
✅ Confirmed exact point of failure  
✅ Learned a new debugging technique (thanks Steven!)  
❌ Hard to separate signal from noise


### 3. Manual Workaround

We'd pinpointed the command that was failing locally, so why not try running it directly from the build server?

I ssh'd into our build server, switched to our deployer user, and ran the [`$_remote_job` command](https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L754-L755) above. What do you know, it worked! The release archive was copied from the build server to our production server, as expected.

I dropped back onto my local machine, commented out the [`upload_release_archive` function](https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L689) from the edeliver script (since the archive was already copied over), and ran the deploy command. Success!

So now things are getting interesting. When I run the command from my local as my `kate` user, it fails. When I run it from our build server as `deployer`, it succeeds. Conclusion: something's messed up with my user.

**Outcome:**  
✅ We found a usuable (albeit super manual) workaround  
✅ Confirmed problem wasn't with our project's edeliver config  
❌ Still can't deploy from my local


### 4. SSH Sanity Check

We'd narrowed down the issue to my user, so next step was to check my ssh config. I confirmed that my keys were on the required servers by ssh'ing in as my user... no problems there. Then I compared my `.ssh/config` file to my teammates' config... nothing out of wack there either.

I also tried deploying after removing everything from the following config files (testing each in isolation, one-by-one), still with zero success:  
- `~/.ssh/known_hosts`
- `~/.bashrc`
- `~/.profile`

**Outcome:**  
✅ Confirmed problem wasn't with my ssh config  
❌ Running out of ideas


## The Turning Point

At this point, desperation was setting in. My teammates and I were stumped, and we'd run out of things to try.

Enter Andres from [OzoneOps](https://ozoneops.com/), our devops contractor. I'd sent him Steven's and my output logs, and he was the first to notice a pretty important difference: in my output, the archive filename had a bunch of weird characters in it.

```bash
# Filename from Steven's output
my_app_X.X.X-XXX-XXXXXXX.release.tar.gz

# Filename from Kate's output
my_app_\033[4;38m\033[KX.X.X-XXX-XXXXXXX\033[m\033[K.release.tar.gz
```

What was going on here? Where were those extra characters coming from?

A quick Google search revealed that these are terminal escape sequences, or [ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code). The terminal normally interprets these sequences as functions, not characters, so you can use them to format output, like adding color to the output of `grep` or `ls` commands.

I like me some colors in my terminal output, so way back in the day (like, two and a half years ago), I added settings to my `.bash_profile` to colorize my `ls` and `grep` output.

```bash
export LSCOLORS=gxxxxxxxcxxxxxcxcxgxgx
export GREP_OPTIONS='--color=always'
export GREP_COLOR='4;38'
```

I'd never given these settings a second thought... until now. Turns out they've been a ticking time bomb, just waiting for this specific scenario.


## The Explanation

The settings above work their colorizing magic by adding ANSI escape codes to output from `ls` and `grep` commands. For example, let's run `grep` on `example.txt` below.

```bash
$ cat example.txt
abcdef
```

```bash
$ grep abc example.txt
abcdef

$ grep --color=always abc example.txt
abcdef
```

Seems harmless, right? The escape codes are properly "escaped", interpretted as functions instead of characters... EXCEPT when your terminal doesn't know how to interpret the sequence. For example, look what happens when you pipe colorized `grep` output into `less`:

```bash
$ grep --color=always abc example.txt | less
ESC[01;31mabcESC[00mdef
(END)
```

Here's the problem. If you pipe colorized output into a function like `less` that doesn't know how to interpret it, the escape codes get treated as if they're regular ol' characters. Same thing happens if you're colorizing `ls` or `grep` in a terminal that doesn't support whatever escape codes you've set through `LSCOLORS` or `GREP_COLOR` variables (see this [helpful post](https://unix.stackexchange.com/questions/143684/what-is-the-problem-with-the-output-of-plink) for a longer explanation).

And that's what was happening to me in the [edeliver](https://github.com/edeliver/edeliver) deploy script.

Looking back at the [edeliver source code](https://github.com/edeliver/edeliver/blob/master/libexec/erlang#L662), the archived release file name is built from the output of `ls` and `grep` commands. Because I had those colorization variables set in my `.bash_profile`, my machine added the color escape codes to the output, which were "misinterpretted" as extra characters in the release archive filename, breaking the deploy.

✋  
.  
.  
.  
🎤 ::mike drop::


## The Solution

Remove any `ls` or `grep` color settings from your `.bash_profile`:

```bash
# ☠️☠️☠️ incompatible with edeliver deploy script ☠️☠️☠️
export LSCOLORS=gxxxxxxxcxxxxxcxcxgxgx
export GREP_OPTIONS='--color=always'
export GREP_COLOR='4;38'
```

## Learnings

Debugging is hard, especially when you're working in a new codebase and/or new language. One thing that can really save you is good error messaging, something Elixir does well and [edeliver can do better](https://github.com/edeliver/edeliver/issues/80).

Next steps from here will be to raise this issue for [edeliver](https://github.com/edeliver/edeliver) and try to contribute some error message improvements (and maybe a fix for this escape sequence nonsense... think we'll just need to disable the color settings before running the deploy command).


## Resources

- [Official edeliver Debugging Tips](https://github.com/edeliver/edeliver#help)
- [More on ANSI escape codes](https://en.wikipedia.org/wiki/ANSI_escape_code)
- [More on GNU `grep` environment variables](https://www.gnu.org/software/grep/manual/html_node/Environment-Variables.html)
- [Similar issue reported on edeliver](https://github.com/edeliver/edeliver/issues/14)
- [More details on edeliver error messaging](https://github.com/edeliver/edeliver/issues/80)
