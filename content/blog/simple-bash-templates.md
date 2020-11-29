---
title: "Simple Bash Templating"
date: 2020-11-28T00:20:21-07:00
tags: [shell, scripts]
draft: false 
---

I needed to set up a project template which would be used to rapidly create many repositories with essentially the
 same skeleton boilerplate. I didn't need much more than simple find replace, and I didn't need the robust features 
 of an entire templating engine -- just something simple with little dependencies. No one will use a template if it 
 takes hours to set up an environment to populate it. In my search, I came across great gnu util called 
 [envsubst](https://www.gnu.org/software/gettext/manual/html_node/envsubst-Invocation.html) which will replace any bash 
 environment variables in a file and output the result to standard out. While similar functionality can be achieved
 with `sed`, the possibility of writing a bad regex and potentially mangling someone's files does not seem like a
 good time.

Since `envsubst` seemed to fit my needs, required no configuration, and was trivial to install, I moved ahead with it
 and began writing a script to populate my template files with exported environment variables. However, I quickly came 
 across a problem -- `envsubst` doesn't support in place substitution, so the following results in an empty file:

```
envsubst < "$file" > "$file"
```

My initial thought was to simply use `tee` to solve my troubles: 

```
envsubst < "$file" | tee "$file"
```

While this command usually works, pipes are asynchronous so sometimes `tee` will truncate a file and `envsubst` will
 read the empty file. This script must be reliable and dependable when executed on someone else's machine. And since it 
 must loop over an entire tree of template files, the chances of failure were too high.

The Solution: Use a tmp file.
```
tmpfile=$(mktemp)
cp -p "./file" "$tmpfile" # Maintain Permissions and Portability
envsubst < "./file" > "$tmpfile" &&  mv "$tmpfile" "./file"
```

It's not graceful or elegant, but it works. Performance isn't a primary concern. 

A practical example of this in action would be to write a yaml with some template variables:

```
---
config:
  setting: default
  secondSetting: ${VAR)
  thirdSetting: ${ANOTHER_VAR}
```

This template config could be filled out with environment variables in the following script:
```
#!/usr/bin/env bash

export VAR=foo
export ANOTHER_VAR=bar

tmpfile=$(mktemp)
cp -p "./config.yml" "$tmpfile"
envsubst < "./config.yml" > "$tmpfile" &&  mv "$tmpfile" "/config.yml"
```

Resulting output config:
```
---
config:
  setting: default
  secondSetting: foo
  thirdSetting: bar
```

Happy templating!


*This is my take on the [Stack Overflow question](https://stackoverflow.com/questions/35078753/can-envsubst-not-do-in-place-substitution)
 on the topic.*

