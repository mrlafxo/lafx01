+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 28 -> Level 29"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 35
+++

**Level Goal**

There is a **git** repository at **ssh://bandit28-git@localhost/home/bandit28-git/repo**. The password for the user bandit28-git is the same as for the user bandit28.

## Solution ##

Let's create a directory with write permissions and clone the repo:

{{< highlight bash>}}
bandit28@bandit:~$ mkdir /tmp/git_level29
bandit28@bandit:~$ cd /tmp/git_level29
bandit28@bandit:/tmp/git_level29$ git clone ssh://bandit28-git@localhost/home/bandit28-git/repo
{{< /highlight>}}

Under the `repo` directory there is a `README` file:

{{< highlight bash>}}
bandit28@bandit:/tmp/git_level29/repo$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: xxxxxxxxxx
{{< /highlight>}}

One useful git command is **log**, that show the commit logs.

{{< highlight bash>}}
bandit28@bandit:/tmp/git_level29/repo$ git log
commit edd935d60906b33f0619605abd1689808ccdd5ee
Author: Morla Porla <morla@overthewire.org>
Date:   Thu May 7 20:14:49 2020 +0200

    fix info leak

commit c086d11a00c0648d095d04c089786efef5e01264
Author: Morla Porla <morla@overthewire.org>
Date:   Thu May 7 20:14:49 2020 +0200

    add missing data

commit de2ebe2d5fd1598cd547f4d56247e053be3fdc38
Author: Ben Dover <noone@overthewire.org>
Date:   Thu May 7 20:14:49 2020 +0200

    initial commit of README.md
{{< /highlight>}}

There is a commit before the last one that we can check with the **checkout** command:

{{< highlight bash>}}
bandit28@bandit:/tmp/git_level29/repo$ git checkout c086d11a00c0648d095d04c089786efef5e01264
{{< /highlight>}}

Opening the `README` file:

{{< highlight bash>}}
bandit28@bandit:/tmp/git_level29/repo$ cat README.md
# Bandit Notes
Some notes for level29 of bandit.

## credentials

- username: bandit29
- password: bbc96594b4e001778eee9975372716b2
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 29** !
