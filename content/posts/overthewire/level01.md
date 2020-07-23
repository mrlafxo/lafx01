+++
draft = false
date = 2020-05-06T17:25:41+02:00
title = "Level 0 -> Level 1"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 7
+++

**Level Goal**

The password for the next level is stored in a file called *readme* located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port *2220*) to log into that level and continue the game.

**Short Lesson**

Useful commands:

>The 'ls' command displays a list of names of all files in the current directory.

>The 'cat' command displays the content of a file on the terminal, when the file name is provided along with the command.


## Solution ##

Starting from the previous ssh session initiated at Level 0, we can find the password for the Level 1 simply listing the current directory and reading the content of the *readme* file!

{{< highlight html >}}
bandit0@bandit:~$ ls
readme
bandit0@bandit:~$ cat readme
boJ9jbbUNNfktd78OOpsqOltutMc3MY1  <- this will be the password for Level 1
{{< /highlight >}}
