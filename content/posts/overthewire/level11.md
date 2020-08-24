+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 10 -> Level 11"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 17
+++

**Level Goal**

The password for the next level is stored in the file **data.txt**, which contains base64 encoded data.


## Solution ##

**Short Lesson**

> In computer science, **base64** is a group of binary-to-text encoding schemes that represent binary data in an ASCII string format by translating it into a radix-64 representation.

> The **base64** command is useful to base64 encode/decode data and print to standard output.

We have to base64 decode the given file, so we will use the `-d` switch (decode):

{{< highlight bash >}}
bandit10@bandit:~$ cat data.txt | base64 -d
The password is IFukwKGsFW8MOq3IRFqrxE1hxTNEbUPR
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 11** !
