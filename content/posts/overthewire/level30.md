+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 29 -> Level 30"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 36
+++

**Level Goal**

There is a git repository at **ssh://bandit29-git@localhost/home/bandit29-git/repo**. The password for the user bandit29-git is the same as for the user bandit29.

## Solution ##

We have to download first the repo in a directory where we have write permissions:

{{< highlight bash>}}
bandit29@bandit:~$ mkdir /tmp/bandit_level30
bandit29@bandit:~$ cd /tmp/bandit_level30
bandit29@bandit:/tmp/bandit_level30$ git clone ssh://bandit29-git@localhost/home/bandit29-git/repo
{{< /highlight>}}

Inside the folder there is a **README** file:

{{< highlight bash>}}
bandit29@bandit:/tmp/bandit_level30$ cd repo
bandit29@bandit:/tmp/bandit_level30/repo$ ls
README.md
bandit29@bandit:/tmp/bandit_level30/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: <no passwords in production!>
{{< /highlight>}}

One git command that is useful here is the **branch** command.

**Short Lesson**

> A **branch** represents an independent line of development. Branches serve as an abstraction for the edit/stage/commit process. You can think of them as a way to request a brand new working directory, staging area, and project history. New commits are recorded in the history for the current branch, which results in a fork in the history of the project.

Let's check the current branch:

{{< highlight bash>}}
bandit29@bandit:/tmp/bandit_level30/repo$ git branch
* master
{{< /highlight>}}

What branches are available?

{{< highlight bash>}}
bandit29@bandit:/tmp/bandit_level30/repo$ git branch -a
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/dev
  remotes/origin/master
  remotes/origin/sploits-dev
{{< /highlight>}}

We have no password in the current branch, which is **master**. Let's access the **dev** branch:

{{< highlight bash>}}
bandit29@bandit:/tmp/bandit_level30/repo$ git checkout dev
Branch dev set up to track remote branch dev from origin.
Switched to a new branch 'dev'
{{< /highlight>}}

And open the **README** file:

{{< highlight bash>}}
bandit29@bandit:/tmp/bandit_level30/repo$ cat README.md
# Bandit Notes
Some notes for bandit30 of bandit.

## credentials

- username: bandit30
- password: 5b90576bedb2cc04c86a9e924ce42faf
{{< /highlight >}}


&nbsp;

We successfully found the password for **Level 30** !
