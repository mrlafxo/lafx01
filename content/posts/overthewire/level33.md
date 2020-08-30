+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 32 -> Level 33"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 39
+++

**Level Goal**

After all this git stuff its time for another escape. Good luck!

## Solution ##

After we login in, we are presented with a shell named UPPERCASE SHELL:

{{< highlight html>}}
WELCOME TO THE UPPERCASE SHELL
>>
{{< /highlight>}}

We have to find a way to escape the current shell!

When we issue a command we are presented with a line which is telling us which environment variables are currently set:

{{< highlight html>}}
WELCOME TO THE UPPERCASE SHELL
>> ls
sh: 1: LS: not found
{{< /highlight>}}

We have three variables:

- `sh` or **$0** is the first one and it is the sh shell
- 1 or **$1**
- `LS` or **$2** which holds the command executed

Let's try to access the first one:

{{< highlight html>}}
>> $0
$
{{< /highlight>}}

We can see we are now in the `sh` shell! From here we can set the **$SHELL** variable:

{{< highlight html>}}
$ export SHELL=/bin/bash
$ echo $SHELL
/bin/bash
{{< /highlight>}}

In order to access the regular bash shell:

{{< highlight html>}}
$ $SHELL
bandit33@bandit:~$
{{< /highlight>}}

From here we can see the password for the next level! Escape completed!

{{< highlight html>}}
bandit33@bandit:~$ cat /etc/bandit_pass/bandit33
c9c3199ddf4121b10cf581a98d51caee
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 33** !

For now, this is the **last level** of the game!
