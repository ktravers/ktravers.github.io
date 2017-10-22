---
layout: post
title: Refash Your Bash - Bash Prompt Customization
---

When I was just starting out learning to code, it quickly became clear I'd be spending _a lot_ of time in Terminal. Never one to skimp on workspace feng shui - just ask my old art world colleagues - I set about post haste to fine tune the CRUD out of my bash profile, starting with the bash prompt. 

Now, I'll caution, there's an endless plethora of resources out there on the ol' internet about customizing your bash prompt, and my first time through, I think I read nearly all of them (really went down the `.bash_profile` rabbit hole, I'll admit). To save you some time and tedium, I'll summarize my specific mods below, with tl:dr links to resources for further customization. 

_Note for fellow rookies:_ My bash prompt is contained inside a nice little function at the top of my `.bash_profile` file. I'd recommend putting this function towards the top since 1) your terminal loads the prompt first and 2) that way it won't interfere with any of the rest of your bash code below. Now without further ado ...

###My Complete Bash Prompt Function

```bash
function parse_git_branch {
  git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
  }

function prompt {
  # DISPLAY GIT STATUS
  export GIT_PS1_SHOWDIRTYSTATE=true
  export GIT_PS1_SHOWUNTRACKEDFILES=true
  
  # LOCAL COLORS
  local       CYAN='\e[0;36m'
  local    ON_CYAN='\e[46m'   # cyan background
  local      BLACK='\e[0;30m'
  local      WHITE='\e[0;37m' 
  local     PURPLE='\e[0;35m'
  local       CHAR="⚡"
  
  # ASCII CHARACTERS
  # ♥ ☆ ⚡ ☕  
  
  export GIT_PS1_SHOWDIRTYSTATE=true
  export GIT_PS1_SHOWUNTRACKEDFILES=true

  # PS1 VARIABLE (PROMPT DISPLAY TEXT)
  export PS1='\n\e[0;30m\e[46m\t\e[0;36m \u @\h: \W\e[0;35m$(__git_ps1)\e[0;37m\n⚡ '
  }
prompt
```

###Result:
![Ktravers Bash Prompt]({{ site.baseurl }}/images/posts/bash-prompt.png "Travers' Bash Prompt")


Not bad, huh?  
Let's dive into the build below.

  
----
###Active Git Branch in Prompt

```bash
function parse_git_branch {
    git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
  }
  
  #more code...
  
  export GIT_PS1_SHOWDIRTYSTATE=true
  export GIT_PS1_SHOWUNTRACKEDFILES=true
```

First up - the function `parse_git_branch` and subsequent `GIT_PS1_SHOW...` lines of code make my programming life infinitely easier by showing me my current git branch and git status (aka DIRTYSTATE) right there in my prompt. Now I always know what branch I'm on, as well as whether I have any tracked/untracked changes, staged commits, etc. 

_Note #1:_ Place the two `export GIT_PS1_SHOW...1` lines of code inside your prompt function, just above your `export PS1=...` line. 

_Note #2:_ You may need to have Git Bash Completion installed as well, which you can activate by also including the code below if you're a homebrew user like me.

```bash
if [ -f `brew --prefix`/etc/bash_completion ]; then
  . `brew --prefix`/etc/bash_completion
  GIT_PS1_SHOWDIRTYSTATE=true
  GIT_PS1_SHOWUNTRACKEDFILES=true
fi
```

  
----
###Prompt Colors 

```bash 
  # LOCAL COLORS
  local       CYAN='\e[0;36m'
  local    ON_CYAN='\e[46m'   # cyan background
  local      BLACK='\e[0;30m'
  local      WHITE='\e[0;37m' 
  local     PURPLE='\e[0;35m'
  local       CHAR="⚡"
```

I'm partial to the dark PRO theme for OSX Terminal, and I used the colors above to end up with a cyan prompt, purple git branch / status, and white command line text. Check out [this source](https://wiki.archlinux.org/index.php/Color_Bash_Prompt#List_of_colors_for_prompt_and_Bash) for more color options.


  
----
###ASCII Characters

```
♥ ☆ ★ ☼ ► 𝄞 ☠ 
```

You can add some real crazy flair to your prompt with ASCII/Unicode characters. Check out a full list [here](http://unicode-table.com/en/).


  
----
###Prompt Escape Codes

_Per the [Bash Manual](http://www.gnu.org/software/bash/manual/bashref.html#Controlling-the-Prompt):_ Bash allows prompt strings to be customized by inserting a number of backslash-escaped special characters. I've listed the ones I've used in my prompt below. Here's a [link to the full list](http://www.gnu.org/software/bash/manual/bashref.html#Controlling-the-Prompt).

`\e`  =>  an ASCII escape character (033)  
`\h`  =>  the hostname up to the first '.'  
`\H`  =>  the hostname  
`\n`  =>  newline  
`\t`  =>  the current time in 24-hour HH:MM:SS format  
`\u`  =>  the username of the current user  
`\W`  =>  the basename of the current working directory, with $HOME abbreviated with a tilde  
`\$`  =>  if the effective UID is 0, a #, otherwise a $  
`\\`  =>  a backslash  
`\[`  =>  begin a sequence of non-printing characters, which could be used to embed a terminal control sequence into the prompt
`\]`  =>  end a sequence of non-printing characters  


  
----
###Summary


**Full prompt:**

```bash
PS1='\n\e[0;30m\e[46m\t\e[0;36m \u @\h: \W\e[0;35m$(__git_ps1)\e[0;37m\n⚡ '
```

**Components:**  

```bash
\n\e[0;30m\e[46m\t
```
new line, current time (24h) in black text with cyan background  

```bash
\e[0;36m \u @\h: \W
``` 
cyan username, @home directory short name:, basename of current working directory (with spaces)  

```bash
\e[0;35m$(__git_ps1)
``` 
purple git branch / status  

```bash
\e[0;37m\n⚡ 
```
all text after git branch will be white, new line with ⚡ unicode character  


_Note:_ The ⚡ character functions like the '$' in the default prompt, so all your commands start with '⚡ _command text_'


  
----
###Further Resources:

- [Lifehacker](http://lifehacker.com/202042/ask-lifehacker--how-do-i-customize-my-command-line-prompt) - nice starting point for customization
- [Unicode Character Tables](http://unicode-table.com/en/) - complete list of all available Unicode characters
- [Bash Manual](http://www.gnu.org/software/bash/manual/bashref.html) - the complete Bash manual, for the real completists
- [ArchLinux Wiki: List of Bash Prompt Colors](https://wiki.archlinux.org/index.php/Color_Bash_Prompt#List_of_colors_for_prompt_and_Bash)
- [AskApache: List of Prompt Escape Codes](http://www.askapache.com/linux/bash-power-prompt.html#Prompt_Escape_Codes)


Thanks for reading, and feel free to share your sweet bash prompts in the comments below!
  

