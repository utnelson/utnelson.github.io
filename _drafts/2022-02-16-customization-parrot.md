---
layout: post
title:  "Customization bash + tmux"
summary: small guide to get a custom shell and intro to tmux
author: utnelson
date: '2022-02-16 22:31:23'
category: ['generall','linux']
thumbnail: \assets\img\posts\customization\shell.png
keywords: shell, linux, parrot, bash, tmux, multiplexer, customization
permalink: /blog/customization/
---

# Intro

The basic shell looks great but I like it clean simple. So I decided to take a look into .bashrc customization. Further more I looked into the tmux possibilities.

## Bash

Customizing the bash prompt is quiet easy. You only need to modify the ~/.bashrc file. It's usefull to backup the file, in case you destroy anything you easily restore your settings.

```console
Creates a backup file
$ cp ~/.bashrc ~/.bashrc.bak 
```

There are a lot of generators out there which create your final code so you don't have to understand all of the code inside. I just wanted to have a visual change in the command line.

I used this one [Easy Bash PS1 Generator](https://ezprompt.net/). 

Because I like it simple, the username + path to current dir is what I want to see. Add some spaces and a dollar sign and we are done. 
My final arrangement looks like this:

![image](\assets\img\posts\customization\arrangement.PNG)

Resulting code:

`export PS1="\[\e[32m\]\u\[\e[m\] [\[\e[34m\]\w\[\e[m\]] \\$[ "`

Now copy and paste this code to the end of your .bashrc file.

Final result looks like this:

![shell](\assets\img\posts\customization\shell.png)

## Tmux

installation

`sudo apt-get install tmux `

manual
```console
$ man tmux

tmux is a terminal multiplexer: it enables a number of terminals to be
created, accessed, and controlled from a single screen.  tmux may be de‐
tached from a screen and continue running in the background, then later
reattached.
```



