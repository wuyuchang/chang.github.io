---
title: Config web server with Nginx & PHP7 in Linux for beginner
---

Both of below are basic command line, If you are not familiar with command line in Linux, you should learn it first.

## Update & Upgrade
``` bash
$ apt-get update
$ apt-get upgrade
```
You must curious what's difference between [update][apt-get] and [upgrade][apt-get].
> **update**
Used to re-synchronize the package index files from their sources. The indexes of available packages are fetched from the location(s) specified in /etc/apt/sources.list(5). An update should always be performed before an upgrade or dist-upgrade.
**upgrade**
Used to install the newest versions of all packages currently installed on the system from the sources enumerated in /etc/apt/sources.list(5). Packages currently installed with new versions available are retrieved and upgraded; under no circumstances are currently installed packages removed, nor are packages that are not already installed retrieved and installed. New versions of currently installed packages that cannot be upgraded without changing the install status of another package will be left at their current version. An update must be performed first so that apt-get knows that new versions of packages are available.

[apt-get]: https://linux.die.net/man/8/apt-get

## Htop, Vim & Screen
``` bash
$ apt-get install htop
$ apt-get install vim
$ apt-get install screen
```
These packages can improve your effective.
***htop:*** [htop][htop] command like [top][top] command, both of these are interactive process viewer for Unix systems. Its help you manage process and monitor resource easy. But [htop][htop] is more convenient than [top][top].
> - In 'htop' you can scroll the list vertically and horizontally to see all processes and complete command lines.
- In 'top' you are subject to a delay for each unassigned key you press (especially annoying when multi-key escape sequences are triggered by accident).
- 'htop' starts faster ('top' seems to collect data for a while before displaying anything).
- In 'htop' you don't need to type the process number to kill a process, in 'top' you do.
- In 'htop' you don't need to type the process number or the priority value to renice a process, in 'top' you do.
- 'htop' supports mouse operation, 'top' doesn't
- 'top' is older, hence, more used and tested.
From http://hisham.hm/htop/index.php?page=comparison

***vim:*** [vim][vim] is almost a proper superset of [vi][vi], everything that is in vi is available in vim.

***screen:***
> 1. use multiple shell window from a single SSH session
2. Keep a shell active even through network disruptions
3. Disconnect and re-connect to a shell sessions from multiple locations
4. Run a long running process without maintaining an active shell session

**Usage**
screen uses the command `ctrl-a` that's the control key and a lowercase "a" as a signal to send commands to screen instead of the shell.
*start*: `screen`
*creating*: `Ctrl-a c`
*switch*: `Ctrl-a n` for the next window or `Ctrl-a p` for the previous window.
*detach*: `Ctrl-a d`
*reattach*: `Ctrl-a r`
*logging*: `Ctrl-a h`
*lock*: `Ctrl-a x`
*stop*: `Ctrl-a k`

## Group & User
``` bash
$ groupadd
$ groupdel
$ useradd
$ passwd
$ usermod
$ userdel
```

### useradd
user add will auto create group that the name same as user
``` bash
$ useradd chang # craete user "chang"
$ useradd -d /data/projects chang # create user "chang" into /data/projects
$ useradd -u 999 -g 998 chang # create user with 999 for user id and 998 for group id
$ useradd -G admings,webadmin,developers chang #
$ useradd -M chang # add user "chang" without home directory
$ useradd -e 2017-01-01 chang # add user "chang" expire at 2017-01-02
$ chage -l chang # show/change user or password expire date
$ useradd -e 2017-01-01 -f 45 chang # password expire after 2017-01-01
$ useradd -c "Yuchang Wu" chang # add user with "Yuchang Wu" infomation
$ tail -1 /etc/passwd # print last 1 line of the file "/etc/passwd"
```

### the meaning of files that user and group saved
- /etc/passwd – User account information.
- /etc/shadow – Secure account information.
- /etc/group – Group account information.
- /etc/gshadow – Secure group account information.
- /etc/login.defs – Shadow password suite configuration..

/etc/passwd format:
username:password:uid:gid:userInfo:homeDirectory:shell

/etc/group format:
groupname:password:gidd:groupMembers

### usermod
for more detail, to below to see
```bash
$ usermod --help
```
``` bash
$ usermod -G www chang # add www group to chang, Be careful, it will remove all existing groups that user belongs, so always add the '-a' with '-G' to append new groups
$ usermod -G www -a chang
```

## Tip
``` bash
$ man order # to see detail of the order
$ apt-cache search package # search package
$ cat /etc/passwd # show all users
$ getent passwd # show all users
$ getent group # show all groups
$ getent group | grep username
$ id username # show username information
$ groups username # show which groups does username have
```

> Ref:
*screen* https://www.rackaid.com/blog/linux-screen-tutorial-and-how-to/#starting
*useradd* http://www.tecmint.com/add-users-in-linux/
*usermod* http://www.tecmint.com/usermod-command-examples/

[htop]: https://hisham.hm/htop/index.php
[top]: https://linux.die.net/man/1/top
[vim]: http://www.vim.org/
[vi]: http://ex-vi.sourceforge.net/
