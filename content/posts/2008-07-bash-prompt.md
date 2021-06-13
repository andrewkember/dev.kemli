---
title: "Bash prompt"
date: 2008-07-29T19:28:08Z
draft: false
# disclaimer: represent|removed
aliases:
    - /articles/bash-prompt/
---

I always forget how to make my bash prompt just the way I like it, so here it is for posterity. In `~/.bashrc`:<!--more-->

```bash
PS1='\[\e]2;\u@\H \w\a\]\[\e[32m\][\t] \[\e[33m\]\w\[\e[0m\]\n\$ '
```

On Ubuntu:

```bash
PS1="\[$(tput bold)\]\[$(tput setaf 1)\][\h]\[$(tput setaf 2)\][\t] \[$(tput setaf 3)\]\w\[$(tput setaf 7
)\]\n\\$ \[$(tput sgr0)\]"
```

This gives me a prompt like this with the time and path:

<div style="background-color:#000"><br>
<code style="color:#0a0;background-color:inherit">[11:00:48]</code><code style="color:#aa0;background-color:inherit"> ~/code/database_info</code><br>
<code style="color:#aaa;background-color:inherit">$ </code>
</div>

And a terminal title like this:

```bash
username@host path
```

[Here's the long guide](http://tldp.org/HOWTO/Bash-Prompt-HOWTO/) to the codes, and [~~here's the one I use from IBM.~~](http://www.ibm.com/developerworks/linux/library/l-tip-prompt/)
