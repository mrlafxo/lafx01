+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 19 -> Level 20"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 26
+++

**Level Goal**

To gain access to the next level, you should use the **setuid binary** in the home directory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.

## Solution ##

**Short Lesson**

> The Unix access rights flags **setuid** and **setgid** (short for "set user ID" and "set group ID") allow users to run an executable with the file system permissions of the executable's owner or group respectively and to change behaviour in directories. They are often used to allow users on a computer system to run programs with temporarily elevated privileges in order to perform a specific task.

We can confirm that the file we are presented with is a **setuid binary** because of the "s" flag in the permission set:

{{< highlight html >}}
bandit19@bandit:~$ ls -l
total 8
-rwsr-x--- 1 bandit20 bandit19 7296 May  7 20:14 bandit20-do
{{< /highlight >}}

Let's try to execute the file as suggested:

{{< highlight html >}}
bandit19@bandit:~$ ./bandit20-do
Run a command as another user.
Example: ./bandit20-do id
{{< /highlight >}}

Essentially we are able to run any command on the system passing it like an argument of the `./bandit20-do` file. Being the owner of the file the `bandit20` user, that means that we will be able to execute commands with the privileges of the `bandit20` user.

Let's read the content of the `/etc/bandit_pass/bandit20` file in order to read the password for the next level:

{{< highlight html >}}
bandit19@bandit:~$ ./bandit20-do cat /etc/bandit_pass/bandit20
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 20** !
