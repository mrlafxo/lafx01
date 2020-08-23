+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 2 -> Level 3"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 9
+++

**Level Goal**

The password for the next level is stored in a file called **spaces in this filename** located in the home directory.


## Solution ##

In order to access file or directories with spaces in their name, we have to 2 ways:


1. Specifying the file / dir name in quotation marks:

{{< highlight bash >}}
bandit2@bandit:~$ cat "spaces in this filename"
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
{{< /highlight >}}

2. Escaping the space in the file / dir name:

{{< highlight bash >}}
bandit2@bandit:~$ cat spaces\ in\ this\ filename
UmHadQclWmgdLOKQ3YNgjWxGoRMb5luK
{{< /highlight >}}


&nbsp;

We successfully found the password for Level 3!
