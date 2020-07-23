+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 1 -> Level 2"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 8
+++

**Level Goal**

The password for the next level is stored in a file called - located in the home directory.

**Short Lesson**

>Every command in Linux may be followed by one or more options. An options is passed to a command with the ' - ' character.

>For example, to also list all the hidden files in the current directory, we can use:

> {{< highlight bash >}}
ls -a
{{< /highlight >}}


## Solution ##

We may be tempted to simply enter the following:

{{< highlight html >}}
bandit1@bandit:~$ cat -
{{< /highlight >}}

But this will not work because the terminal will try to interpret the ' - ' character as an attempt to pass an option.
In order to read a file beginning with the dash character, we can use the *cat* command in 2 interesting ways:

1. Specifying the path of the file
{{< highlight bash >}}
bandit1@bandit:~$ cat ./-
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
{{< /highlight >}}
Here `./` stands for current directory.



2. Using input redirection
{{< highlight bash >}}
bandit1@bandit:~$ cat <- -
CV1DtqXWVFXTvM2F0k09SHz0YwRINYA9
{{< /highlight >}}  

&nbsp;

We successfully found the password for Level 2!
