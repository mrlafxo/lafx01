+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 14 -> Level 15"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 21
+++

**Level Goal**

The password for the next level can be retrieved by submitting the password of the current level to **port 30000 on localhost**.

## Solution ##

We have to connect on port 30000 on the current machine. We can do that with the **nc** command.

> **netcat** is a simple Unix utility which reads and writes data across network connections, using TCP or UDP protocol. It is designed to be a reliable  "back-end"
  tool  that  can be used directly or easily driven by other programs and scripts.
  At the same time, it is a feature-rich network debugging and  exploration  tool,
  since it can create almost any kind of connection you would need and has several
  interesting built-in capabilities.

{{< highlight html >}}
bandit14@bandit:~$ nc localhost 30000
4wcYUJFw0k0XLShlDzztnTBHiqxU3b3e
Correct!
BfMYroe26WYalil77FoDi9qh59eK5xNr
{{< /highlight >}}

Once we establish the connection, we have to insert the password for the current level. The service listening on port 30000 will respond with the password for the next level.

&nbsp;

We successfully found the password for **Level 15** !
