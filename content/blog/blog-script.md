---
title: "Hugo Markdown Setup"
date: 2018-03-03T15:57:18-07:00
draft: false 
tags: [scripts, shell, config]
---
Now that I've got my blog set up I thought I should share the set up I use script I use when writing blog posts (and maybe markdown pages in general).

If you don't know what Hugo is, I suggest you [check it out](https://gohugo.io). At its core a static site generator that takes markdown files and compiles them into static html files, but it does so much more. Packaged with the file compiler is a webserver that uses websockets to support a live reload feature. What this means in practice is that if you have a text editor that has an auto save feature, you can simply start up the server and start writing a post and watch it reload
in a broswer window. 

My text editor of choice is [Neovim](https://neovim.io/), which is a refactor of Vim. I like the ideas behind the project and have yet to come across any downsides of it compared to Vim. In order to enable autosave started in the editor, I open a new file and while I'm in command mode enter:

```autocmd TextChanged,TextChangedI <buffer> silent write```

My server start of script is still in development
