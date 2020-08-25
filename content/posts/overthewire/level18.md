+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 17 -> Level 18"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 24
+++

**Level Goal**

There are 2 files in the homedirectory: **passwords.old and passwords.new**. The password for the next level is in **passwords.new** and is the only line that has been changed between **passwords.old and passwords.new**.

NOTE: if you have solved this level and see ‘Byebye!’ when trying to log into bandit18, this is related to the next level, bandit19.

## Solution ##

Basically we have to find the differences between the two files provided. The **diff** command will help with the task:

**Short Lesson**
> On Unix-like operating systems, the **diff** command analyzes two files and prints the lines that are different. In essence, it outputs a set of instructions for how to change one file to make it identical to the second file.

We will use the `-y` switch to present the result in columns:

{{< highlight html >}}
bandit17@bandit:~$ diff -y passwords.new passwords.old

....
bCk5uJggdb5TkvEkxwDLSkJuwdGzZpfU				bCk5uJggdb5TkvEkxwDLSkJuwdGzZpfU
1sh2JnXE1KcChopsVJ0nxMYsrpsp2Auw				1sh2JnXE1KcChopsVJ0nxMYsrpsp2Auw
GwlKel7OwVXQG9FqIQntI0BAHea0IWBD				GwlKel7OwVXQG9FqIQntI0BAHea0IWBD
sydIUj42mUfYK9xw1S9aPPB72rgagnxh				sydIUj42mUfYK9xw1S9aPPB72rgagnxh
pcpkwztEjxg5EK0HABjmEvGUSCSdQW4F				pcpkwztEjxg5EK0HABjmEvGUSCSdQW4F
LlomcOUT6d7lA2cJrYhCEhCChKCPrRao				LlomcOUT6d7lA2cJrYhCEhCChKCPrRao
kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd			|	w0Yfolrc5bwjS4qw5mq1nnQi6mF03bii
TVzFbgWpqUPE4fwAJPCz4rT7GemAZUjz				TVzFbgWpqUPE4fwAJPCz4rT7GemAZUjz
WCETP1i90TZJSbKZ24ly5rhNKva8sSdy				WCETP1i90TZJSbKZ24ly5rhNKva8sSdy
....
{{< /highlight >}}

The different lines are highlighted with the "|" character.
So the password for the next level will be: `kfBf3eYk5BPBRzwjqutbbfE887SVc5Yd`.


&nbsp;

We successfully found the password for **Level 18** !
