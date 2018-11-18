---
title: "Starting Task Warrior"
date: 2018-03-28T19:19:28-06:00
tags: [shell,taskwarrior, shell, task-warrior]
draft: false
---

I've recently started using [Task Warrior](https://taskwarrior.org/), a command line application that helps to track and manage your todo list. 

With simple commands like `task ls`, `task add Task Name Here`, and `task 10 done`, you can list, add, and complete
tasks in your task list, but the application is built with flexibility in mind. There's an optional backend server which 
serves an api that the cli and 3rd party applications can utilize, allowing cross platform task syncing across machines 
and devices.

That being said, I don't run a a server for my Task Warrior, mainly because I use a single work station and 
keep my work todos there. To improve the flow I have with using task warrior, I've installed an [Oh-My-Zsh](https://github.com/robbyrussell/oh-my-zsh) plugin
which helps with tab completion and some basic aliases.

Installation will vary based on your setup, but on OSX it was as simple as calling brew: `brew install task`
and then adding a line to my Oh-My-Zsh plugins list:
```text
 plugins=(
    taskwarrior
 )
```
Once completed, restarting my shell was all I needed to do to get started.

Give Task Warrior a try and get used to spending a little more time in your command line and a little less time waiting for web 
pages and applications to load.
