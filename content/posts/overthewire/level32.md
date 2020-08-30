+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 31 -> Level 32"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 38
+++

**Level Goal**

There is a git repository at **ssh://bandit31-git@localhost/home/bandit31-git/repo**. The password for the user bandit31-git is the same as for the user bandit31.

Clone the repository and find the password for the next level.

## Solution ##

We have to download first the repo in a directory where we have write permissions:

{{< highlight bash>}}
bandit31@bandit:~$ mkdir /tmp/bandit_level32
bandit31@bandit:~$ cd /tmp/bandit_level32
bandit31@bandit:/tmp/bandit_level32$ git clone ssh://bandit31-git@localhost/home/bandit31-git/repo
{{< /highlight>}}

Inside the folder there is a **README** file:

{{< highlight bash>}}
bandit31@bandit:/tmp/bandit_level32/repo$ cat README.md
This time your task is to push a file to the remote repository.

Details:
    File name: key.txt
    Content: 'May I come in?'
    Branch: master
{{< /highlight>}}

So in order to pass this level we have to push our own file to the remote repository. If we check the content of the **repo** folder, we can see that there is a **.gitignore** file.

**Short Lesson**

> A **.gitignore** file specifies intentionally untracked files that Git should ignore.

{{< highlight html>}}
bandit31@bandit:/tmp/bandit_level32/repo$ cat .gitignore
*.txt
{{< /highlight>}}

This file is telling that all .txt file should be untracked. We do not that to happen because we have to push our own .txt file to the remote repo. So we can delete it!

{{< highlight html>}}
bandit31@bandit:/tmp/bandit_level32/repo$ rm .gitignore
{{< /highlight>}}

Let's now create the file named **key.txt**:

{{< highlight html>}}
bandit31@bandit:/tmp/bandit_level32/repo$ echo "May I come in?" > key.txt
{{< /highlight>}}

And push it on the remote repo:

{{< highlight html>}}
bandit31@bandit:/tmp/bandit_level32/repo$ git add key.txt
bandit31@bandit:/tmp/bandit_level32/repo$ git commit -m "adding a new file"
[master 359ffe2] adding a new file
 1 file changed, 1 insertion(+)
 create mode 100644 key.txt
bandit31@bandit:/tmp/bandit_level32/repo$ git push
Could not create directory '/home/bandit31/.ssh'.
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is SHA256:98UL0ZWr85496EtCRkKlo20X3OPnyPSB5tB5RPbhczc.
Are you sure you want to continue connecting (yes/no)? yes
Failed to add the host to the list of known hosts (/home/bandit31/.ssh/known_hosts).
This is a OverTheWire game server. More information on http://www.overthewire.org/wargames

bandit31-git@localhost's password:
Counting objects: 3, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 323 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: ### Attempting to validate files... ####
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
remote: Well done! Here is the password for the next level:
remote: 56a9bf19c63d650ce78e6ec0354ee45e    <-- password for level 32 -->
remote:
remote: .oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.oOo.
remote:
To ssh://localhost/home/bandit31-git/repo
 ! [remote rejected] master -> master (pre-receive hook declined)
error: failed to push some refs to 'ssh://bandit31-git@localhost/home/bandit31-git/repo'
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 32** !
