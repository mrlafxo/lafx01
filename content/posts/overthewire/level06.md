+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 5 -> Level 6"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 12
+++

**Level Goal**

The password for the next level is stored in a file somewhere under the **inhere** directory and has all of the following properties:

- human-readable
- 1033 bytes in size
- not executable

## Solution ##

**Short Lesson**

> The **find** command in UNIX is a command line utility for walking a file hierarchy. It can be used to find files and directories and perform subsequent operations on them. It supports searching by file, folder, name, creation date, modification date, owner and permissions. By using the ‘-exec’ other UNIX commands can be executed on files or folders found.

The inhere directory contains different directories:

{{< highlight bash >}}
bandit5@bandit:~/inhere$ ls -l
total 80
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere00
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere01
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere02
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere03
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere04
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere05
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere06
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere07
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere08
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere09
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere10
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere11
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere12
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere13
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere14
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere15
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere16
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere17
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere18
drwxr-x--- 2 root bandit5 4096 May  7 20:15 maybehere19
{{< /highlight >}}


We have to use the **find** command combined with options based upon the required properties:

{{< highlight bash >}}
bandit5@bandit:~/inhere$ find . -type f -size 1033c ! -executable
./maybehere07/.file2

bandit5@bandit:~/inhere$ cat ./maybehere07/.file2
DXjZPULLxYr17uwoI01bNLQbtFemEgo7
{{< /highlight >}}

Note that **!** stands for a negation and **1033c** stands for **1033 bytes**.

&nbsp;

We successfully found the password for **Level 6** !
