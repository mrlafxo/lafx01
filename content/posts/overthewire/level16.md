+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 15 -> Level 16"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 22
+++

**Level Goal**

The password for the next level can be retrieved by submitting the password of the current level to **port 30001 on localhost** using SSL encryption.

Helpful note: Getting “HEARTBEATING” and “Read R BLOCK”? Use -ign_eof and read the “CONNECTED COMMANDS” section in the manpage. Next to ‘R’ and ‘Q’, the ‘B’ command also works in this version of that command…

## Solution ##

This level is the same as the previous one, except for the fact that we have to connect using **SSL**.

The **nc** utility does not support SSL, so we can use its variant **Ncat**, that was originally developed for the Nmap project, along with the `--ssl` switch. 

{{< highlight html >}}
ncat localhost 30001 --ssl
BfMYroe26WYalil77FoDi9qh59eK5xNr
Correct!
cluFn7wTiGryunymYOu4RcffSxQluehd
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 16** !
