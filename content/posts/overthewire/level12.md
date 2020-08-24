+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 11 -> Level 12"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 18
+++

**Level Goal**

The password for the next level is stored in the file **data.txt**, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions.

## Solution ##

If we try to read the content of the file **data.txt** we can see that the message has been encoded:

{{< highlight bash >}}
bandit11@bandit:~$ cat data.txt
Gur cnffjbeq vf 5Gr8L4qetPEsPk8htqjhRK8XSP6x2RHh
{{< /highlight >}}

The level description suggests that we are dealing with a **ROT13** (rotate by 13 places) substitution cipher.

**Short Lesson**

> **ROT13** ("rotate by 13 places", sometimes hyphenated ROT-13) is a simple letter substitution cipher that replaces a letter with the 13th letter after it, in the alphabet. ROT13 is a special case of the Caesar cipher which was developed in ancient Rome.
Because there are 26 letters (2×13) in the basic Latin alphabet, ROT13 is its own inverse; that is, to undo ROT13, the same algorithm is applied, so the same action can be used for encoding and decoding. The algorithm provides virtually no cryptographic security, and is often cited as a canonical example of weak encryption.

> ![ROT13](/images/bandit/ROT13.png)

In order to solve this level we will use the **tr** command, which is a command line utility for translating or deleting characters. It supports a range of transformations including uppercase to lowercase, squeezing repeating characters, deleting specific characters and basic find and replace.

{{< highlight bash >}}
bandit11@bandit:~$ cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
The password is 5Te8Y4drgCRfCx8ugdwuEX8KFC6k2EUu
{{< /highlight >}}

Each character in the first set of `tr` will be replaced with the corresponding character in the second set. E.g. A replaced with N, B replaced with O, etc.. And then the same for the lower case letters.

&nbsp;

We successfully found the password for **Level 12** !
