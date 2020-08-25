+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 20 -> Level 21"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 27
+++

**Level Goal**

There is a **setuid** binary in the home directory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).

**NOTE**: Try connecting to your own network daemon to see if it works as you think.

## Solution ##

Let's first open a port for receiving connections:

{{< highlight html >}}
bandit20@bandit:~$ nc -lnvp 4444
listening on [any] 4444 ...
{{< /highlight >}}

Now we have to open a new terminal and a new SSH connection to the same machine, and from here we will be using the suid binary specified in the level description to connect to the 4444 port:

{{< highlight html >}}
bandit20@bandit:~$ ./suconnect 4444
{{< /highlight >}}

We will see on the first terminal window the connection received. From here it is just a matter of pasting the current level password, and the new password will be presented:

{{< highlight html >}}
bandit20@bandit:~$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [127.0.0.1] from (UNKNOWN) [127.0.0.1] 46024
GbKksEFF4yrVs6il55v6gwY5aVje5f0j
gE269g2h3mw3pwgrj0Ha9Uoqen1c9DGr
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 21** !
