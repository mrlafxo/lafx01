+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 25 -> Level 26"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 32
+++

**Level Goal**

Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not **/bin/bash**, but something else. Find out what it is, how it works and how to break out of it.

## Solution ##

Under the working directory we have a SSH Private Key that we can use to connect to the next level:

{{< highlight html >}}
bandit25@bandit:~$ ls
bandit26.sshkey
bandit25@bandit:~$ ssh -i bandit26.sshkey localhost -l bandit26 -t /bin/sh
Could not create directory '/home/bandit25/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:98UL0ZWr85496EtCRkKlo20X3OPnyPSB5tB5RPbhczc.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit25/.ssh/known_hosts).
This is a OverTheWire game server. More information on http://www.overthewire.org/wargames

  _                     _ _ _   ___   __  
 | |                   | (_) | |__ \ / /  
 | |__   __ _ _ __   __| |_| |_   ) / /_  
 | '_ \ / _` | '_ \ / _` | | __| / / '_ \
 | |_) | (_| | | | | (_| | | |_ / /| (_) |
 |_.__/ \__,_|_| |_|\__,_|_|\__|____\___/
Connection to localhost closed.
{{< /highlight>}}

But the connection is closed because the shell for the Level 26 is not **/bin/bash**.

Let's find out what kind of shell is used for that level. That kind of info is present in the **/etc/passwd** file:

{{< highlight html>}}
bandit25@bandit:~$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
.........
bandit23:x:11023:11023:bandit level 23:/home/bandit23:/bin/bash
bandit24:x:11024:11024:bandit level 24:/home/bandit24:/bin/bash
bandit25:x:11025:11025:bandit level 25:/home/bandit25:/bin/bash
bandit26:x:11026:11026:bandit level 26:/home/bandit26:/usr/bin/showtext
bandit27:x:11027:11027:bandit level 27:/home/bandit27:/bin/bash
........
{{< /highlight>}}

Let's inspect the content of **/usr/bin/showtext**:

{{< highlight html>}}
bandit25@bandit:~$ cat /usr/bin/showtext
#!/bin/sh

export TERM=linux

more ~/text.txt
exit 0
{{< /highlight>}}

The script is opening the text.txt file with the **more** command.

**Short Lesson**

> The **more** command is used to view the text files in the command prompt, displaying one screen at a time in case the file is large (for example log files). The more command also allows the user do scroll up and down through the page.

It can also be used in the so called `command view`, from where we can execute commands. We have just to reduce the terminal window and execute the previous command to connect to the next level.

![Level25_More](/images/bandit/Level25_More.png)

From here we have to press press `v` to edit the file. Now we can read the password for the next level entering the following command:

`:e /etc/bandit_pass/bandit26`

![Level25_Password](/images/bandit/Level25_Password.png)

So the password for Level 26 is: `5czgV9L3Xx8JPOyRbXh6lQbmIOWvPT6Z`.

But if we try to use it to connect to the next level, it wont work! Why is that?
That is because we have to set the `SHELL` variable for the next level, and we can do that from the same `command view` used before.

Let's press `v` to edit the file like we did before and then:

- `:set shell=/bin/bash`
- `:shell`

![Level25_WHOAMI](/images/bandit/Level25_WHOAMI.png)

We are logged in as the user **bandit26**!

**Note**: Next level will assume that we are in this shell already. 

&nbsp;

We successfully accessed the **Level 26** !
