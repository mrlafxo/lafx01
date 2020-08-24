+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 8 -> Level 9"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 15
+++

**Level Goal**

The password for the next level is stored in the file **data.txt** and is the only line of text that occurs only once.

## Solution ##

The provided file is a collection of multiple passwords. Some of the passwords are repeated through the file. Our goal is to find the password that appears only one.
We will be using two new commands, **sort** and **uniq**.

**Short Lesson**

> The **sort** command is used to sort a file, arranging the records in a particular order. By default, the sort command sorts file assuming the contents are ASCII. Using options in sort command, it can also be used to sort numerically.

> The **uniq** command in Linux is a command line utility that reports or filters out the repeated lines in a file.
In simple words, uniq is the tool that helps to detect the adjacent duplicate lines and also deletes the duplicate lines.

First of all we have to sort the file, in order to 'group' all of the occurrences of the same password.

{{< highlight bash >}}
bandit8@bandit:~$ cat data.txt | sort
{{< /highlight >}}

After that we have to find the password that appears only once. The `uniq` command has a switch `(-u)` that prints only unique lines. So we can pipe the `uniq` command to the previous one like:

{{< highlight bash >}}
bandit8@bandit:~$ cat data.txt | sort | uniq -u
UsvVyFSfZZWbi6wgC7dAFyFuR6jQQUhR
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 9** !
