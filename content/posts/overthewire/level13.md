+++
draft = false
date = 2020-05-06T17:26:01+02:00
title = "Level 12 -> Level 13"
description = ""
slug = ""
tags = []
categories = []
externalLink = ""
series = []
weight = 19
+++

**Level Goal**

The password for the next level is stored in the file **data.txt**, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work using mkdir. For example: mkdir /tmp/myname123. Then copy the datafile using cp, and rename it using mv (read the manpages!)

## Solution ##

**Short Lesson**

> In computing, a **hex dump** is a hexadecimal view of computer data, from RAM or from a computer file or storage device. Looking at a hex dump of data is usually done in the context of either debugging or reverse engineering.
In a hex dump, each byte (8-bits) is represented as a two-digit hexadecimal number. Hex dumps are commonly organized into rows of 8 or 16 bytes, sometimes separated by whitespaces. Some hex dumps have the hexadecimal memory address at the beginning and/or a checksum byte at the end of each line.

Printing the file content on screen we can see that the file is, as said, a hex dump:

{{< highlight bash >}}
bandit12@bandit:~$ cat data.txt
00000000: 1f8b 0808 0650 b45e 0203 6461 7461 322e  .....P.^..data2.
00000010: 6269 6e00 013d 02c2 fd42 5a68 3931 4159  bin..=...BZh91AY
00000020: 2653 598e 4f1c c800 001e 7fff fbf9 7fda  &SY.O...........
00000030: 9e7f 4f76 9fcf fe7d 3fff f67d abde 5e9f  ..Ov...}?..}..^.
....
....
{{< /highlight >}}

In Linux, the command to make a hexdump or the reverse, is called **xxd**.
In this case, we want to first to the reverse `(-r)` of the hex dump, using the following command:

{{< highlight bash >}}
xxd -r data.txt
{{< /highlight >}}

But we do not have write permissions on the current folder, so we have to create a new directory under `/tmp` as suggested.

{{< highlight bash >}}
bandit12@bandit:~$ mkdir /tmp/bandit_level_12
bandit12@bandit:~$ cd /tmp/bandit_level_12
{{< /highlight >}}

Now we can do the reverse of the hex dump on the file and redirect the output to a new file called `data` under the new folder we just created:

{{< highlight bash >}}
bandit12@bandit:~$ xxd -r data.txt > /tmp/bandit_level_12/data
{{< /highlight >}}

We move to the `/tmp` directory and check the file type of the new `data` file:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: gzip compressed data, was "data2.bin", last modified: Thu May  7 18:14:30 2020, max compression, from Unix
{{< /highlight >}}

We can see that the file is a `gzip` compressed. We have to decompress it, but before doing so we have to rename it and give it a `.gz` extension in order to be able to pass it to the `gzip` command:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data data.gz
{{< /highlight >}}

Decompressing the file:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ gzip -d data.gz
{{< /highlight >}}

After that we check the file type of the new file obtained decompressing the previous one:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: bzip2 compressed data, block size = 900k
{{< /highlight >}}

The new data file is a `bzip2` compressed file.

Like before, in order to decompress it, we have to rename the file to the proper extension and then decompress it with the right program:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data data.bz2
bandit12@bandit:/tmp/bandit_level_12$ bzip2 -d data.bz2
{{< /highlight >}}

We obtain a new file:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: gzip compressed data, was "data4.bin", last modified: Thu May  7 18:14:30 2020, max compression, from Unix
{{< /highlight >}}

Let's decompress it like done before with `gzip`:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data data.gz
bandit12@bandit:/tmp/bandit_level_12$ gzip -d data.gz
{{< /highlight >}}

We now obtain a new file which appears to be a `TAR archive`:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: POSIX tar archive (GNU)
{{< /highlight >}}

Let's rename the file with a `.tar` extension and decompress it:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data data.tar
bandit12@bandit:/tmp/bandit_level_12$ tar xf data.tar
{{< /highlight >}}

Where `xf` stands for `extract file`.

We now have a new file called `data5.bin` which is a new `TAR archive`:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data5.bin
data5.bin: POSIX tar archive (GNU)
{{< /highlight >}}

Let's decompress this one too:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data5.bin data.tar
bandit12@bandit:/tmp/bandit_level_12$ tar xf data.tar
{{< /highlight >}}

A new file `data6.bin` is generated, which is a `bzip2` compressed file:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data6.bin
data6.bin: bzip2 compressed data, block size = 900k
{{< /highlight >}}

Let's decompress it like we did before:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data6.bin data.bz2
bandit12@bandit:/tmp/bandit_level_12$ bzip2 -d data.bz2
{{< /highlight >}}

A new file and `TAR archive` is created:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: POSIX tar archive (GNU)
{{< /highlight >}}

Once again:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data data.tar
bandit12@bandit:/tmp/bandit_level_12$ tar xf data.tar
{{< /highlight >}}

A new file is created which appears to be a gzip compressed file:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data8.bin
data8.bin: gzip compressed data, was "data9.bin", last modified: Thu May  7 18:14:30 2020, max compression, from Unix
{{< /highlight >}}

Let's decompress this one too:

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ mv data8.bin data.gz
bandit12@bandit:/tmp/bandit_level_12$ gzip -d data.gz
{{< /highlight >}}

The new file created is a ASCII file and finally we are able to read the password for the next level!

{{< highlight bash >}}
bandit12@bandit:/tmp/bandit_level_12$ file data
data: ASCII text
bandit12@bandit:/tmp/bandit_level_12$ cat data
The password is 8ZjyCRiBWFYkneahHwxCv3wb2a1ORpYL
{{< /highlight >}}

&nbsp;

We successfully found the password for **Level 13** !
