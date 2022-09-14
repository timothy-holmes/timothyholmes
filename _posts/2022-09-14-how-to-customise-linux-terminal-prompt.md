---
title: How To Customise Linux Terminal Prompt
---

{% raw %}

I'm using a very low powered laptop. When I fire up Docker containers, they can take a while. I'd like to know how long, but I didn't check when I started it. If the terminal prompt had a timestamp then I wouldn't need to.

It might be kinda useful info to have. Naturally, Im going to spent multiple hours to finding and implementing the solution. Here's what I currently get:

```bash
<user>@<host>:<current directory>$
```

Here's what I want:

```bash
[<YYYY-MM-DD-HH-MM-SS-TZ>]<user>@<host>:<full path>
$
```

How to change it in three simple steps. _*I have to confess it took me a couple of hours; hours I could have spent on worthy causes such as housework, cooking et al._

1. Modify the PS1 environment variable in the `.bashrc` file found in your home directory. Use `man bash` (the PROMPTING section) or [bashrcgenerator.com](http://bashrcgenerator.com).
2. Complicating things, `.bashrc` already has a few lines involved in setting the PS1 var. Figure was `$debian_chroot` is, and why you why want to leave it in the terminal prompt. [chrooted debian?](https://askubuntu.com/questions/372849/what-does-debian-chrootdebian-chroot-do-in-my-terminal-prompt)
3. Take the time to learn about `tput` and add colour commands. [tput?,](https://linuxcommand.org/lc3_adv_tput.php) [256 colours](https://www.ditig.com/256-colors-cheat-sheet)

Here's what was in there already:

```bash
if [ "$color_prompt" = yes ]; then
    PS1='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[00m\]\$ '
else
    PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '
fi
```

And here's my first attempt:

```bash
BG_WHITE="$(tput setab 15)"
BG_BLACK="$(tput setab 0)"
FG_NAVY="$(tput setaf 4)"
FG_TEAL="$(tput setaf 6)"
FG_PURPLE="$(tput setaf 5)"
FG_RED="$(tput setaf 9)"
FG_BOLD="$(tput bold)"
DEFAULT_ATTRS="$(tput sgr0)"

if [ "$color_prompt" = yes ]; then
    PS1="${debian_chroot:+${BG_BLACK}${FG_WHITE}${FG_BOLD}($debian_chroot)}${DEFAULT_ATTRS}"
    PS1+="${BG_WHITE}${FG_RED}${FG_BOLD}\[\D{%Y-%m-%d-%H:%M:%S}\]"
    PS1+="${FG_PURPLE}\u@\h"
    PS1+="${DEFAULT_ATTRS}${BG_SILVER}${FG_RED}:"
    PS1+="${FG_TEAL}${FG_BOLD}\w\n"
    PS1+="${FG_RED}\$ ${DEFAULT_ATTRS}"
else
    PS1="${debian_chroot:+($debian_chroot)}"
    PS1+="\[\D{%Y-%m-%d-%H:%M:%S}\]"
    PS1+="\u@\h"
    PS1+=":"
    PS1+="\w\n"
    PS1+="\$ "
fi
```

_Caveat 1:_ The time in the prompt is the time the _last_ command _finished_ rather than the time when the current command _started_

_Caveat 2:_ I don't know where to find where colour preferences have been set, so I'm hardcoding them here. It might look odd if I changed my colour preferences

{% endraw %}
