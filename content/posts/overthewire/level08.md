+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 7 -> Level 8"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 14
+++

**Level Goal**

The password for the next level is stored in the file **data.txt** next to the word **millionth**.

## Solution ##

**Short Lesson**

> The **grep** command is used to search text. It searches the given file for lines containing a match to the given strings or words. It is one of the most useful commands on Linux and Unix-like system.

The given file is composed by a hundreds of lines containing a word followed by a password. The password we are searching for stands next to the world "millionth". We can solve this level piping the content of the file with the **grep** command:

{{< highlight bash >}}
bandit7@bandit:~$ cat data.txt | grep millionth
millionth	cvX2JJa4CFALtqS87jk27qwqGhBM9plV
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 8** !
