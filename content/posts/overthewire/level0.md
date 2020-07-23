+++
draft = false
date = 2020-03-30T18:50:25+02:00
title = "Level 0"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 6
+++

**Level Goal**

The goal of this level is for you to log into the game using SSH. The host to which you need to connect is *bandit.labs.overthewire.org*, on port *2220*. The username is *bandit0* and the password is *bandit0*.

Once logged in, go to the Level 1 page to find out how to beat Level 1.

**Short Lesson**

What is SSH?

>Secure Shell (SSH) is a cryptographic network protocol for operating network services
>securely over an unsecured network. Typical applications include remote command-line,
>login, and remote command execution, but any network service can be secured with SSH.
>SSH provides a secure channel over an unsecured network in a client–server architecture,
>connecting an SSH client application with an SSH server.
>The protocol specification distinguishes between two major versions, referred to as
>SSH-1 and SSH-2. The standard TCP port for SSH is 22.


## Solution ##

The following is the syntax used to connect to a SSH server:

{{< highlight bash >}}
ssh user@IP -p port_number
{{< /highlight >}}

In our case, to connect to Level 0, we launch the terminal command:

{{< highlight bash >}}
ssh bandit0@bandit.labs.overthewire.org -p 2220
{{< /highlight >}}

We will be prompted with a password request. Insert it, and the connection will be confirmed!
This is the main task of this level, understanding how to initiate an ssh connection.

In the next chapter, starting from this session, we'll see how to gain the password for the Level 1!
