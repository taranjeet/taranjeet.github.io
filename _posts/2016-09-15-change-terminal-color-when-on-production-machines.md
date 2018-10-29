---
layout: post
title: Change Terminal color when on production machines.
date: 2016-09-15 22:00
tag:
- bash
category: blog
author: taranjeet
description: This post is about how to change color when on production machines.
---

Many times we are faced with the requirement of sshing into production machines, looking up something from production database
or debugging the app. Sometimes the issues are so serious, that we totally forget that we are on production machines and we do some
serious breaking shit over there.

So one of the way to clearly differentiate whether you are on a production machine or local can be changing the color of the terminal.
Yes, it is possible and it works wonderfully.

This post is about doing such configuration on Iterm2, oh-my-zsh and Mac(Os X El Captain). Ideally, this will remain same for most of the
linux systems(but having oh-my-zsh is a requirement as I am using its dot folder).

To begin with, first we will create different color profiles on Iterm 2.

* Open Iterm2 prefences (shortcut key ⌘ + ,)
* Go to Profiles tab ![iterm2-preferences](/public/img/iterm2-pref.png)
* Click on the plus (+) icon and add profiles. You can customize font color, etc in profile or choose a predefined color theme. In my case I have created three profiles namely _Default_, _Production_ and _Staging_. Each profile is having a different color theme.

* Assuming that oh-my-zsh is installed, cd into `~/.oh-my-zsh/custom` and create a file named `iTerm2-ssh.zsh`.

```
cd ~/.oh-my-zsh/custom
vim iTerm2-ssh.zsh
```

The contents of the file will be something like

```
function tabc() {
  NAME=$1; if [ -z "$NAME" ]; then NAME="Default"; fi
  echo -e "\033]50;SetProfile=$NAME\a"
}

function tab-reset() {
    NAME="Default"
    echo -e "\033]50;SetProfile=$NAME\a"
}

function colorssh() {
    if [[ -n "$ITERM_SESSION_ID" ]]; then
        trap "tab-reset" INT EXIT
        if [[ "$*" =~ "prod*" ]]; then
            tabc Production
        elif [[ "$*" =~ "stage*" ]]; then
            tabc Staging
        else
            tabc Other
        fi
    fi
    ssh $*
}
compdef _ssh tabc=ssh

alias ssh="colorssh"
```

How is this working?

* colorssh is the function which is responsible for changing color. So it is aliased to _ssh_.
* _tabc_ is the function responsible for selecting a _Profile_. When it is passed no arguments, then
it selects the `Default` profile.
* `\033]` is `Esc`, `\033]50;` means Escape 50 character. This is a custom escape code, which iTerm2 supports
to change profile on the fly.
* tab-reset is responsible for restoring the theme to default whenever any ssh connection is terminated. trap is
something like event listener, which traps(or listen) any int exit code. Whenever any ssh session is terminated,
exit code is produced. This traps that exit code and calls `tab-reset`, thereby resetting the profile to `Default`.
* Normal if else blocks command line argument to regular expression bases string. `$*` is the expansion of positional
parameters(starting from $1). `=~` is something similar like `==` and `!=` and it does regular expression based matching.
* compdef is for auto completion. ssh, scp are all completed by the function `_ssh`. Here the line means that function `tabc`
is to be completed just like ssh.


References:
* [coderwall](https://coderwall.com/p/t7a-tq/change-terminal-color-when-ssh-from-os-x)
