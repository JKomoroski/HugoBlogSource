---
title: "Hosting Version 99 Does Not Exist"
date: 2023-03-19T14:11:00-07:00
tags: [java, maven]
draft: false
---

### TLDR;

I'm now hosting my own [version99](https://version99.komoro.ski).

###

Version 99 Does Not Exist originated from 
[this blog post](http://day-to-day-stuff.blogspot.com/2007/10/announcement-version-99-does-not-exist.html) but there are
some issues with the original site. It does not support head requests which modern versions of maven can make. It also 
doesn't have https hosting from it's creator which modern versions of maven really try to enforce. The pom files it would
generate wouldn't correctly rename group id url paths to group id package paths. Finally a small issue is that the landing
page it would generate didn't support relative links to the current host.

I corrected all of those issues and published a version to my quay.io as a docker container for quick deployments. The 
quickest way to run version 99 is to run my docker image:
```
docker run -p 8080:8080 quay.io/jared_kom/version99:v0.0.1
```

We'll see if upstream accepts my prs and if they put the original version 99 behind a secure connection.

Links:
- Original Version 99 Host: [http://version99.grons.nl/]()
- Original Version 99 Repo: [https://github.com/erikvanoosten/version99]()
- Docker Container: [https://quay.io/repository/jared_kom/version99?tab=tags]()
- My Fork Repo: [https://github.com/JKomoroski/version99]()
