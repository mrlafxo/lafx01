+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 18 -> Level 19"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 25
+++

**Level Goal**

The password for the next level is stored in a file **readme** in the home directory. Unfortunately, someone has modified **.bashrc** to log you out when you log in with SSH.

## Solution ##

When we try to login to level 18 with the previously discovered password, we are informed that the connection is closed.

{{< highlight html >}}
Byebye !
Connection to bandit.labs.overthewire.org closed.
{{< /highlight >}}

This is why the **.bashrc** file was modified and is not letting to connect.

But what is the **.bashrc** file?

> The **.bashrc** file is a script that is executed whenever a new terminal session is started in interactive mode. This is what happens when you open a new terminal window or just open a new terminal tab.
By contrast a terminal session in login mode will ask you for user name and password and execute the **~/.bash_profile** script. This is what takes place, for instance, when you log on to a remote system through SSH.
The **.bashrc** file itself contains a series of configurations for the terminal session. This includes setting up or enabling: colouring, completion, the shell history, command aliases and more.

We are not able to connect via the bash shell, but the **ssh** command has an option `(-t)` to let us connect via the **sh shell**:

{{< highlight html >}}
ssh bandit.labs.overthewire.org -p2220 -l bandit18 -t /bin/sh
{{< /highlight >}}

We are now connected via **/bin/sh** and we can check that echoing the **$TERM** variable, which contains the current used terminal:

{{< highlight html >}}
$ $TERM
/bin/sh: 4: xterm-256color: not found
{{< /highlight >}}

We can now read the password for the next level on the **readme** file:

{{< highlight html >}}
$ cat readme
IueksS7Ubh8G3DCwVzrTd8rAVOwq3M5x
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 19** !
