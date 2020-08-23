+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 4 -> Level 5"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 11
+++

**Level Goal**

The password for the next level is stored in the only human-readable file in the **inhere** directory.
Tip: if your terminal is messed up, try the “reset” command.

## Solution ##

We can solve this level in three ways:

1. With the **file** command:

**Short Lesson**

> The **file** command is used to determine the type of a file. File type may be of human-readable (e.g. ‘ASCII text’) or MIME type(e.g. ‘text/plain; charset=us-ascii’). This command tests each argument in an attempt to categorize it.

We can pass every filename in the inhere directory to the file command and search for the one that is human-readable:

{{< highlight bash >}}
bandit4@bandit:~/inhere$ ls
-file00  -file01  -file02  -file03  -file04  -file05  -file06  -file07  -file08  -file09
bandit4@bandit:~/inhere$ file ./-file00
./-file00: data
bandit4@bandit:~/inhere$ file ./-file01
./-file01: data
bandit4@bandit:~/inhere$ file ./-file02
./-file02: data
bandit4@bandit:~/inhere$ file ./-file03
./-file03: data
bandit4@bandit:~/inhere$ file ./-file04
./-file04: data
bandit4@bandit:~/inhere$ file ./-file05
./-file05: data
bandit4@bandit:~/inhere$ file ./-file06
./-file06: data
bandit4@bandit:~/inhere$ file ./-file07
./-file07: ASCII text
{{< /highlight >}}

The **file07** appears to be the one we were searching for:

{{< highlight bash >}}
bandit4@bandit:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
{{< /highlight >}}

2. Like the first solution, using the **file** command but more efficiently:

The * character is used as a **wildcard** and it substitutes everyting that is in the current directory.

{{< highlight bash >}}
bandit4@bandit:~/inhere$ file ./*
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
{{< /highlight >}}

3. With a more elegant method, combining the **type** and the **file** command:

**Short Lesson**

> Xargs is a great command that reads streams of data from standard input, then generates and executes command lines; meaning it can take output of a command and passes it as argument of another command. If no command is specified, xargs executes echo by default.

First we 'search' all the file in the current directory, then we pass the output to the **file** command via **xargs**:

{{< highlight bash >}}
bandit4@bandit:~/inhere$ find . -type f | xargs file
./-file01: data
./-file00: data
./-file06: data
./-file03: data
./-file05: data
./-file08: data
./-file04: data
./-file07: ASCII text
./-file02: data
./-file09: data
{{< /highlight >}}

Here is the password again:

{{< highlight bash >}}
bandit4@bandit:~/inhere$ cat ./-file07
koReBOKuIDDepwhWk7jZC0RTdopnAYKh
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 5** !
