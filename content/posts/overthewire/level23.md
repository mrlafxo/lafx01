+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 22 -> Level 23"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 29
+++

**Level Goal**

A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

**NOTE**: Looking at shell scripts written by other people is a very useful skill. The script for this level is intentionally made easy to read. If you are having problems understanding what it does, try executing it to see the debug information it prints.

## Solution ##

Let's discover what script is running as part of cron:

{{< highlight bash >}}
bandit22@bandit:~$ cd /etc/cron.d/
bandit22@bandit:/etc/cron.d$ ls
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit24
cronjob_bandit17_root  cronjob_bandit23  cronjob_bandit25_root
bandit22@bandit:/etc/cron.d$ cat cronjob_bandit23
@reboot bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
* * * * * bandit23 /usr/bin/cronjob_bandit23.sh  &> /dev/null
{{< /highlight >}}

What is the **/usr/bin/cronjob_bandit23.sh** doing?

{{< highlight bash >}}
bandit22@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit23.sh
#!/bin/bash

myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)

echo "Copying passwordfile /etc/bandit_pass/$myname to /tmp/$mytarget"

cat /etc/bandit_pass/$myname > /tmp/$mytarget
{{< /highlight>}}

Let's analyze the script:

- the `myname` variable contains the value of the `whoami` command, which will be `bandit23`
- the `md5sum` command calculates the MD5 hash of the string `"I am user bandit22"`, which will be `8ca319486bfbbc3663ea0fbe81326349  -`
- the `cut` command, together with the `-d` (delimiter) and `-f` (select field) switches, will only select the first part of the hash, that is `8ca319486bfbbc3663ea0fbe81326349`. This will also be the value of the `mytarget` variable
- the script is basically redirecting the content of **/etc/bandit_pass/bandit23** to **/tmp/8ca319486bfbbc3663ea0fbe81326349**


If we read the content of **/tmp/8ca319486bfbbc3663ea0fbe81326349**, we will find the password for the next level:

{{< highlight bash >}}
bandit22@bandit:/etc/cron.d$ cat /tmp/8ca319486bfbbc3663ea0fbe81326349
jc1udXuA1tiHqjIsL8yaapX5XIAI6i0n
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 23** !
