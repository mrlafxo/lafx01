+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 3 -> Level 4"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 10
+++

**Level Goal**

The password for the next level is stored in a hidden file in the **inhere** directory.

## Solution ##

In order to list hidden files inside a directory, we must use the `ls` command with the `-a` (all) switch:

{{< highlight bash >}}
bandit3@bandit:~$ cd inhere
bandit3@bandit:~/inhere$ ls -a
.  ..  .hidden
{{< /highlight >}}

Alternatively we could also use the `long listing format (-l)` in conjunction with the `-a` switch, in order to check permissions and to understand if we are dealing with files or directories:

{{< highlight bash >}}
bandit3@bandit:~/inhere$ ls -al
total 12
drwxr-xr-x 2 root    root    4096 May  7 20:14 .
drwxr-xr-x 3 root    root    4096 May  7 20:14 ..
-rw-r----- 1 bandit4 bandit3   33 May  7 20:14 .hidden
{{< /highlight >}}

So .hidden is a file. Let's see its content:

{{< highlight bash >}}
bandit3@bandit:~/inhere$ cat .hidden
pIwrPrtPN36QITSp3EQaw936yaFoFgAB
{{< /highlight >}}

&nbsp;

We successfully found the password for Level 4 !
