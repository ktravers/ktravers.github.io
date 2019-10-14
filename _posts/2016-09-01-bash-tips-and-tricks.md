---
layout: post
title: BASH Tips and Tricks
tags: ['code reading', 'bash']
---

All tips courtesy [@devinburnette](https://github.com/devinburnette)

## Basics

### `$ w`
see who else is on box

```bash
$ w
14:15  up 5 days, 19:23, 7 users, load averages: 1.47 1.56 1.82
USER     TTY      FROM             LOGIN@  IDLE WHAT
ktravers console  -                Mon11   3days -
_mbsetupuser console  -            Mon11   3days -
ktravers s000     -                Mon11   1:35 gulp
ktravers s001     -                Mon11       - w
ktravers s002     -                Mon11   4:10 node /Users/ktravers/.nvm/versions/node/v6.2.2/bin/sails lift
ktravers s004     -                Mon11   3days /usr/local/Cellar/rabbitmq/3.5.1/erts-6.1/bin/../../erts-6.1/bin/beam.smp -W w -K true -A30
ktravers s005     -                Mon11   3days redis-server *:6379
```

### `$ write [user]`
send message to specific user

### `$ wall`
broadcast to all users on box

### `$ whoami`
get your current user

```bash
$ whoami
ktravers
```

### `$ who am i`
verbose get current user

```bash
$ who am i
ktravers     ttys001  Aug 29 11:06
```

### `$ clear`
put command line at top of window

### `$ man [whatevs]`
get manual for [whatevs] command

```bash
$ man man

man(1)                                                                                                                  man(1)

NAME
       man - format and display the on-line manual pages

SYNOPSIS
       man  [-acdfFhkKtwW]  [--path]  [-m system] [-p string] [-C config_file] [-M pathlist] [-P pager] [-B browser] [-H html-
       pager] [-S section_list] [section] name ...
...
```

### `$ history`
readout of bash history

```bash
$ history
397  git log
398  cap production deploy
399  glr
400  glr
401  gbv
402  git branch -d next-lesson-bugfix
403  git branch -d slack-bugfix
404  gco -b slack-bugfix
405  gst
```

### `$ !515`
run bash history command number

### `$ !!`
re-run last bash history command

```bash
$ mkdir foo
$ cd !$
```

make directory and change into it

### `$ cd -`
change into last directory you were in

## Keyboard shortcuts

CTRL + k => delete from cursor to end of line  
CTRL + u => delete from cursor to beginning of line  
CTRL + y => print deleted chars back  
CTRL + s => to go forward in history  
CTRL + r => to go back in history  

## Searching

### `$ grep rails foo`
search

### `$ grep -A2 -B2 rails foo`
search and show before and after

### `$ grep -v rails foo`
everything except `rails`

### `$ grep -i Rails foo`
case insensitive search

### `$ grep -o rails`
match search

### `$ grep -e [expression]` / # `$ egrep [expression]`
pass expression to search

### `$ sed 's/Passenger/Logan/g' foo`
search and replace (where `/` is a delimiter)

### `$ sed -i 's/Passenger/Logan/g' foo`
search and replace case insensitive (where `/` is a delimiter)

### `$ sed '/Logan/d' foo`
search and delete Logan (where `/` is a delimiter)

Note on delimiter:  
`/` is a tricky choice if searching for paths, bc need to escape each `/`
Use `+` instead, for example

### `$ cat foo | awk -F: {print $1}`
get everything before first column

### `$ awk /UID/,/Logan\ Rackapp/ foo`
get everything between two points

### `$ locate ruby`
find every file with "ruby" in its path

Note: has to be turned on =>  
`$ launchctl load -w /System/Library/LaunchDaemons/com.apple.locate.plist`

### `$ find / -iname ruby`
real time recursive lookthrough all system files

### `$ find / -iname ruby -type f -mtime +30`
real time recursive lookthrough all system files  
-type f => only files  
-mtime +30 => return only files modified time older than past 30 days

### `$ find / -iname ruby -type f -mtime +30 -exec ls -l {}`
run execution on all found files

## System

### `$ ps`
current user processes running

```bash
$ ps
  PID TTY           TIME CMD
 1772 ttys000    0:00.35 -bash
55712 ttys000    0:35.49 /Users/ktravers/.rvm/rubies/ruby-2.2.2/bin/ruby bin/rails s
55720 ttys000    0:00.60 npm
55721 ttys000   27:02.15 gulp
 4199 ttys001    0:02.26 -bash
 5028 ttys002    0:00.29 -bash
29345 ttys002    0:04.80 node /Users/ktravers/.nvm/versions/node/v6.2.2/bin/sails lift
 6678 ttys004    0:00.20 -bash
 8845 ttys004    9:44.14 /usr/local/Cellar/rabbitmq/3.5.1/erts-6.1/bin/../../erts-6.1/bin/beam.smp -W w -K true -A30 -P 1048576 -- -root /us
 7487 ttys005    0:00.19 -bash
 8844 ttys005    0:49.27 redis-server *:6379
```

### `$ ps aux`
all processes running

```bash
$ ps aux
USER              PID  %CPU %MEM      VSZ    RSS   TT  STAT STARTED      TIME COMMAND
ktravers        55721  28.8  0.3  3251428  21468 s000  S+   12:34PM  27:09.25 gulp
_windowserver     248   6.7  0.7  3624064  58676   ??  Ss   Fri06PM 197:29.40 /System/Library/Frameworks/ApplicationServices.framework/Frame
_coreaudiod       137   4.8  0.1  2500488   4592   ??  Ss   Fri06PM  13:58.15 /usr/sbin/coreaudiod
ktravers         1672   2.7  3.5  3699432 296196   ??  S    Mon11AM  64:49.15 /Applications/Google Chrome.app/Contents/MacOS/Google Chrome -
ktravers         1673   2.4  1.5  2793764 125056   ??  S    Mon11AM  17:46.66 /Applications/Utilities/Terminal.app/Contents/MacOS/Terminal -
ktravers        54234   1.2  1.5  3485456 128292   ??  S    Wed11AM   6:13.23 /Applications/Google Chrome.app/Contents/Versions/52.0.2743.11
ktravers        18832   0.9  2.0  3685316 168976   ??  S     5:21PM   3:11.80 /Applications/Google Chrome.app/Contents/Versions/52.0.2743.11
ktravers        66895   0.4  3.6  3931612 301060   ??  S    Wed12PM   5:36.17 /Applications/Google Chrome.app/Contents/Versions/52.0.2743.11
ktravers         1742   0.3  3.5  4015008 292568   ??  S    Mon11AM  19:10.98 /Applications/Google Chrome.app/Contents/Versions/52.0.2743.11
ktravers         9924   0.2  3.3  3101936 273696   ??  S     4:20PM   8:15.92 /Applications/Sublime Text.app/Contents/MacOS/Sublime Text
ktravers        67279   0.1  0.0  2471188   3964   ??  S    Tue11AM   2:12.74 /Applications/Utilities/Adobe Creative Cloud/CoreSync/Core Syn
ktravers        55712   0.1  1.9  2701340 158856 s000  S+   12:34PM   0:35.54 /Users/ktravers/.rvm/rubies/ruby-2.2.2/bin/ruby bin/rails s
root              104   0.1  0.0  2473708   2560   ??  Ss   Fri06PM  11:13.25 /usr/libexec/hidd
...
```

### `$ netstat -an`
shows all connected and listening ports and sockets

```bash
$ netstat -an
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp4       0      0  192.168.2.58.54966     54.152.118.174.443     ESTABLISHED
tcp4       0      0  192.168.2.58.54959     52.205.164.104.443     ESTABLISHED
tcp4       0      0  192.168.2.58.54953     104.237.191.1.443      ESTABLISHED
...
```

### `$ wc foo`
word count, line count, character count

### `$ wc -w`
word count only

### `$ netstat -an | grep :22 | wc -l`
port connection count

### `$ lsof`
list open files on system (and what process is keeping them open)

```bash
$ lsof
total 80
drwxr-xr-x  19 ktravers   646 Aug  1 13:06 .
drwxr-xr-x  10 ktravers   340 Aug  8 19:09 ..
drwxr-xr-x  15 ktravers   510 Sep  1 14:55 .git
-rw-r--r--   1 ktravers    90 Aug 20  2015 .gitignore
-rw-r--r--   1 ktravers   336 Aug 20  2015 404.md
-rw-r--r--   1 ktravers  1867 Aug  1 13:06 _config.yml
drwxr-xr-x   5 ktravers   170 Aug 20  2015 _includes
drwxr-xr-x   5 ktravers   170 Aug 20  2015 _layouts
drwxr-xr-x  25 ktravers   850 Sep  1 14:12 _posts
drwxr-xr-x   6 ktravers   204 Aug 20  2015 _scss
-rw-r--r--   1 ktravers   572 Sep 10  2015 about.md
drwxr-xr-x  34 ktravers  1156 Mar  8 17:13 assets
-rw-r--r--   1 ktravers    21 Aug 20  2015 CNAME
-rw-r--r--   1 ktravers   789 Aug 20  2015 feed.xml
drwxr-xr-x   8 ktravers   272 Aug 20  2015 images
-rw-r--r--   1 ktravers   403 Aug 20  2015 index.html
-rw-r--r--   1 ktravers  1077 Aug 20  2015 LICENSE
-rw-r--r--   1 ktravers   521 Aug 20  2015 README.md
-rwxr-xr-x   1 ktravers  3868 Aug 20  2015 style.scss
```

## Network

### `$ ifconfig`

network config on box
