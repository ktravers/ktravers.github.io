---
layout: post
title: Refash' Your Bash - Bash Prompt Customization
---

When I was just starting out learning to code, it was clear from day one that I'd be spending _a lot_ of time in Terminal, so one of the first things I did was learn how to fine tune my bash profile, starting with the bash prompt. 

Now, I'll caution, there's an endless plethora of resources out there on the ol' internet about customizing your bash prompt, and my first time through, I think I read nearly all of them (really went down the `.bash_profile` rabbit hole, I'll admit). To save you some time and tedium, I'll summarize my specific mods below, with links to resources for further customization. 

_Note for fellow rookies:_ My bash prompt is contained inside a nice little function at the top of my `.bash_profile` file. I'd recommend putting the function at the top since 1) your terminal loads the prompt first and 2) it won't interfere with any of the rest of your bash code below. Now without further ado ...

  
----
###My Complete Bash Prompt Function

```
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
![Ktravers Bash Prompt]({{ site.baseurl }}/assets/bash-prompt.png "Travers' Bash Prompt")


Not bad, huh?  
Let's dive into the build below.

  
----
###Active Git Branch in Prompt

``` 
function parse_git_branch {
    git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/ (\1)/'
  }
```

If you're using command line for git, you want your prompt to display your current git branch and status. It's pretty invaluable, all in all. To really kick this into high gear, be sure to include the following two lines of code inside your prompt function, just above your `export PS1=...` line. 

```
export GIT_PS1_SHOWDIRTYSTATE=true
export GIT_PS1_SHOWUNTRACKEDFILES=true
```

_Note:_ You may need to have Git Bash Completion installed as well, which you can activate with the code below if you're a homebrew user like me.

``` 
if [ -f `brew --prefix`/etc/bash_completion ]; then
  . `brew --prefix`/etc/bash_completion
  GIT_PS1_SHOWDIRTYSTATE=true
  GIT_PS1_SHOWUNTRACKEDFILES=true
fi
```

  
----
###Prompt Colors 

``` 
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

```
PS1='\n\e[0;30m\e[46m\t\e[0;36m \u @\h: \W\e[0;35m$(__git_ps1)\e[0;37m\n⚡ '
```

**Components:**  

```
\n\e[0;30m\e[46m\t

``` 
new line, current time (24h) in black text with cyan background  

```
\e[0;36m \u @\h: \W
``` 
cyan username, @home directory short name:, basename of current working directory (with spaces)  

```
\e[0;35m$(__git_ps1)
``` 
purple git branch / status  

```
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
  

