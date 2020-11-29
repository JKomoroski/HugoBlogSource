---
title: "My Zsh Setup"
date: 2018-03-25T14:19:49-06:00
tags: [shell, config, cli]
draft: false 
---

I'm a big fan of my terminal and the cli I use everyday. A big reason for that is the customizations I use in it. I haven't just settled on using the terminal as it comes out of the box, which has made it the most effective and productive tool I use everyday. I thought I should share some of those customizations here on my blog, and this first post is focused on [Zsh](https://en.wikipedia.org/wiki/Z_shell).

To start, I've installed and set Zsh as my default shell. Zsh has many built-in features that make it much easier to use as an interactive terminal, while remaining close to the more posix focused bash. A few examples of the bells and whistles that come standard in Zsh but are missing or limited in Bash are:

* Improved `cd` Tab Completion
* Improved Command Tab Completion
* Path Expansion
* Right Prompt Display
* Spelling Auto-Correct or Prompted Correction
* Global Aliases (aliases that appear anywhere in your command string)
* Extended Globbing Capabilities

If you'd like more detailed documentation on what sets Zsh apart, take a look a the Zsh FAQ [here](http://zsh.sourceforge.net/FAQ/zshfaq02.html). A particularly exciting part of Zsh is the active community behind it. There are several plugin managers, and a host of people building plugins for what seems like every command line tool, and creating more themes than should ever be necessary.

I use the [Oh My Zsh](http://ohmyz.sh/) configuration manager. It helps breakdown my .zshrc (zsh config) file into more manageable pieces. You can see my .zshrc file [here](https://raw.githubusercontent.com/JKomoroski/dotfiles/master/.zshrc). I used the default config they provided as a starting point, but I've slowly added my own aliases and adjusted the agnoster theme to suit my needs over time. I use a vim binding so I can edit lines with vim text navigation. The screen shot below
shows my go to tmux window, running zsh with a scratch pane and my task list open in adjacent panes. I'll have more on tmux and taskwarrior in the near future.  

---

<img src="/img/zsh_view.png" alt="Tmux and Zsh running on iTerm2" class="container"/>

---

That's the intro to my favorite shell. I intend to write a guide to use my setup in the future, but for now I've settled with expressing my passion for the tool I've come to love.
