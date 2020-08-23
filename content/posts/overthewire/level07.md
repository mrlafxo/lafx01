+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 6 -> Level 7"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 13
+++

**Level Goal**

The password for the next level is stored **somewhere on the server** and has all of the following properties:

- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

## Solution ##

Like we did in the previous level, we can use the **find** command:

{{< highlight bash >}}
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c
{{< /highlight >}}

The problem with this command is that the terminal will also print out all the files for which we don't have read permissions. In order to avoid that we can redirect the **standard error output** to **/dev/null**, which is like an empty space where anything gets deleted.

{{< highlight bash >}}
bandit6@bandit:~$ find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
/var/lib/dpkg/info/bandit7.password
{{< /highlight >}}

{{< highlight bash >}}
bandit6@bandit:~$ cat /var/lib/dpkg/info/bandit7.password
HKBPTKQnIay4Fw76bEy8PVxKEDQRKTzs
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 7** !
