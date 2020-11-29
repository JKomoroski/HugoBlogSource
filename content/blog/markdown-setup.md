---
title: "Hugo Markdown Setup"
date: 2018-03-03T15:57:18-07:00
draft: false 
tags: [scripts, shell, config]
---
Now that I've got my blog set up I thought I should share the set up I use script I use when writing blog posts (and maybe markdown pages in general).

If you don't know what Hugo is, I suggest you [check it out](https://gohugo.io). At its core a static site generator that takes markdown files and compiles them into static html files, but it does so much more. Packaged with the file compiler is a webserver that uses websockets to support a live reload feature. What this means in practice is that if you have a text editor that has an auto save feature, you can simply start up the server and start writing a post and watch it reload
in a broswer window. 

My text editor of choice is [Neovim](https://neovim.io/), which is a refactor of Vim. I like the ideas behind the project and have yet to come across any downsides of it compared to Vim. In order to enable autosave in the editor, I open a new file and enter the following while in command mode:

```autocmd TextChanged,TextChangedI <buffer> silent write```

My server start script is still a work in progress. The current form can be viewed in the blog's source code [here](https://raw.githubusercontent.com/JKomoroski/HugoBlogSource/master/start_server). Here's what it looks like today:

```
#!/bin/sh

# Parse options for this script
# Extensable for future additions
BEGIN=true
END=false
while getopts "eh?" option
do
    case "${option}"
        in
        e)
            BEGIN=false
            END=true
            ;;
        h)
            printf '%s\n' "Use flag -e (end) to end development, default behavior starts development."
            ;;
        *)
            printf '%s\n' "Use flag -e (end) to end development, default behavior starts development."
            ;;
    esac
done

if [ $BEGIN == true ]
then
    printf '%s\n' "Beginning Development, Starting Hugo Server" \
    && `hugo server -w -D -F --disableFastRender >> /dev/null &` \
    && printf '%s\n' "Hugo Server Started" \
    && printf '%s\n' "http://localhost:1313"
elif [ $END == true ]
then
    printf '%s\n' "Stopping Hugo Server..." \
    && killall hugo >> /dev/null \
    && printf '%s\n' "Hugo Server Stopped!"
fi
```
My goal with that script is to try to remain posix compliant in the script, so it will be portable to my future machines
 or to workstations other than my own. In brief, the script will start the hugo server in the current directory with a 
 specific set of flags. It has a `-h` flag for help in the event that anyone else clones it and if the script is used 
 with the `-e` flag it will end the current session. It's not the best bash script and there are better ways to do it, 
 but it gets the job done. 

Unfortunately, I can't launch my browser of choice from the script at the time of start up because OSX and firefox 
 don't play nice, so that piece is still manual at this time. However, I do echo the localhost hugo port, and use my 
 terminal "parse for url" feature to simply open the blog on a new page. 

---
---

While getting this set up, I did find typing out the current date/time everytime I started a .md file from
 scratch tedious and annoying. I found this little gem of a command: 

`:put =strftime('%FT%T%z')` 

This outputs:

`2018-03-03T15:57:18-0700`

This does not work with hugo because it is not completely ISO 8601 compliant as the timezone offset should be colon 
 delimited between the hour and minute offset. Your output may vary depending on the implementation of `strftime()` 
 on your system. I found on my Linux system that the shell command `date +%FT%T%:z` works, so calling this from Nvim
 with proper escapes suits me: `r!date "+\%FT\%T\%:z"` . 
 
My system is using a BSD `strftime()` implementation with the had the following gem in the man page:
```
BUGS
     There is no conversion specification for the phase of the moon.
```
