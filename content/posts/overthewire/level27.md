+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 26 -> Level 27"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 33
+++

**Level Goal**

Good job getting a shell! Now hurry and grab the password for bandit27!

## Solution ##

**Note**: This level will assume that we are in this shell obtained in the previous level.

The working directory has a script named **bandit27-do**. Let's see what it does:

{{< highlight html>}}
bandit26@bandit:~$ ./bandit27-do
Run a command as another user.
Example: ./bandit27-do id
{{< /highlight>}}

We can run any command with it, so let's read the password for the next level!

{{< highlight html>}}
bandit26@bandit:~$ ./bandit27-do cat /etc/bandit_pass/bandit27
3ba3118a22e93127a4ed485be72ef5ea
{{< /highlight>}}

&nbsp;

We successfully found the password for **Level 27** !
