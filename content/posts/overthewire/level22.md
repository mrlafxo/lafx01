+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 21 -> Level 22"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 28
+++

**Level Goal**

A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

## Solution ##

**Short Lesson**

> **Cron** allows Linux and Unix users to run commands or scripts at a given date and time. You can schedule scripts to be executed periodically. Cron is one of the most useful tool in a Linux or UNIX like operating systems. It is usually used for sysadmin jobs such as backups or cleaning /tmp/ directories and more. The cron service (daemon) runs in the background and constantly checks the **/etc/crontab** file, and **/etc/cron.*/** directories. It also checks the **/var/spool/cron/** directory.

First we have to list the content of the **/etc/cron.d/** directory:

{{< highlight bash >}}
bandit21@bandit:/$ cd /etc/cron.d/ && ls -l1
total 24
-rw-r--r-- 1 root root  62 May 14 13:40 cronjob_bandit15_root
-rw-r--r-- 1 root root  62 Jul 11 15:56 cronjob_bandit17_root
-rw-r--r-- 1 root root 120 May  7 20:14 cronjob_bandit22
-rw-r--r-- 1 root root 122 May  7 20:14 cronjob_bandit23
-rw-r--r-- 1 root root 120 May 14 09:41 cronjob_bandit24
-rw-r--r-- 1 root root  62 May 14 14:04 cronjob_bandit25_root
{{< /highlight >}}

We are dealing with the level 22, so let's open the associated file:

{{< highlight bash >}}
bandit21@bandit:/etc/cron.d$ cat cronjob_bandit22
@reboot bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
* * * * * bandit22 /usr/bin/cronjob_bandit22.sh &> /dev/null
{{< /highlight >}}

There is a script running every minute. Let's see its content:

{{< highlight html >}}
bandit21@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit22.sh
#!/bin/bash
chmod 644 /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
cat /etc/bandit_pass/bandit22 > /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
{{< /highlight >}}

What is the script doing?

First it sets permissions on the `/tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv` file:

- 4 + 2 = **6** -> Read + Write permissions for the file owner
- **4** -> Read permissions for the group
- **4** -> Read permissions for everybody else

Then it redirects the password from `/etc/bandit_pass/bandit22` on that file.

We can read the password for the next level just opening that file:

{{< highlight html >}}
bandit21@bandit:/etc/cron.d$ cat /tmp/t7O6lds9S0RqQh9aMcz6ShpAoZKF7fgv
Yk7owGAcWjwMVRwrTesJEwB7WVOiILLI
{{< /highlight >}}


&nbsp;

We successfully found the password for **Level 22** !
