---
title: "You Should Use You Should Use"
date: 2018-11-18T13:06:35-07:00
tags: [shell,taskwarrior, ysu, cli]
draft: false
---
A while ago I installed a zsh plugin to remind me of my plugins called 
[You Should Use](https://github.com/MichaelAquilina/zsh-you-should-use) (ysu). The idea behind it is fairly simple, if 
you have configured an alias for a given command, it will will remind you about that alias if you use the command 
directly, rather call it via the alias. For example, when I use task warrior to mark a task completed, I may type:
```
task 6 done
```
Since I have a zsh plugin with aliases for Task Warrior installed, ysu returns the following message before executing 
the command:
 ```
 Found existing alias for "task". You should use: "t"
 ```
This indicates that I could have used `t 6 done` to mark my sixth task as completed, saving a few key strokes. By 
default, the command then executes as normal. The plugin also has a hardcore mode that requires the use of existing 
aliases before executing a command.

Turn your interactive shell into a more inviting place by adding ysu to it today. Start using those aliases you
installed with plugins and forgot about or  or never knew about,