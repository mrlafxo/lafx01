+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 27 -> Level 28"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 34
+++

**Level Goal**

There is a **git** repository at **ssh://bandit27-git@localhost/home/bandit27-git/repo**. The password for the user bandit27-git is the same as for the user bandit27.

## Solution ##

For this level we  have to download a git repository stored online.

**Short Lesson**

> **Git** is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

First we will create a directory under `/tmp` where we have write permissions:

{{< highlight bash>}}
bandit27@bandit:~$ mkdir /tmp/git_level28 && cd /tmp/git_level28
{{< /highlight>}}

From here we will use the **git clone** command:

{{< highlight html>}}
bandit27@bandit:/tmp/git_level28$ git clone ssh://bandit27-git@localhost/home/bandit27-git/repo
Cloning into 'repo'...
Could not create directory '/home/bandit27/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:98UL0ZWr85496EtCRkKlo20X3OPnyPSB5tB5RPbhczc.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit27/.ssh/known_hosts).
This is a OverTheWire game server. More information on http://www.overthewire.org/wargames

bandit27-git@localhost's password:
remote: Counting objects: 3, done.
remote: Compressing objects: 100% (2/2), done.
remote: Total 3 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (3/3), done.
{{< /highlight>}}

The password is stored inside a `README` file in the `repo` directory:

{{< highlight html>}}
bandit27@bandit:/tmp/git_level28$ ls
repo
bandit27@bandit:/tmp/git_level28$ cd repo
bandit27@bandit:/tmp/git_level28/repo$ ls
README
bandit27@bandit:/tmp/git_level28/repo$ cat README
The password to the next level is: 0ef186ac70e04ea33b4c1853d2526fa2
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 28** !
