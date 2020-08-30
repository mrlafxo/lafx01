+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 30 -> Level 31"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 37
+++

**Level Goal**

There is a git repository at **ssh://bandit30-git@localhost/home/bandit30-git/repo**. The password for the user bandit30-git is the same as for the user bandit30.

Clone the repository and find the password for the next level.

## Solution ##

We have to download first the repo in a directory where we have write permissions:

{{< highlight bash>}}
bandit29@bandit:~$ mkdir /tmp/bandit_level31
bandit29@bandit:~$ cd /tmp/bandit_level31
bandit30@bandit:/tmp/bandit_level31$ git clone ssh://bandit30-git@localhost/home/bandit30-git/repo
{{< /highlight>}}

Inside the folder there is a **README** file:

{{< highlight bash>}}
bandit30@bandit:/tmp/bandit_level31/repo$ cat README.md
just an epmty file... muahaha
{{< /highlight>}}

**Short Lesson**

> Like most VCSs, **Git** has the ability to tag specific points in a repository’s history as being important. Typically, people use this functionality to mark release points (v1.0, v2.0 and so on).

Let's see if there is any **tag** in the current repo:

{{< highlight bash>}}
bandit30@bandit:/tmp/bandit_level31/repo/.git$ git tag
secret
bandit30@bandit:/tmp/bandit_level31/repo/.git$ git show secret
47e603bb428404d265f59c42920d81e5
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 31** !
