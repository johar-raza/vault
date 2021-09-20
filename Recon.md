#cheatsheet #pentesterlab
# Recon
---

- robots.txt
- security.txt
- subdomains
- vhost using curl
`curl -H "Host: vhost.<domain>" <URL/IP>`

- vhost brute forcing
`gobuster vhost -u <URL/IP> -w <wordlist> -t 100`

- check if application supports multiple backends due to load balancing (use curl)
	`curl http://www.<URL>.com/?[1-20]` -> 20 requests

	OR if there are multiple query strings in the URL
	`curl http://www.<URL>.com/?myVar=111&fakeVar=[1-20]`

- Use dig to get TXT records and CNAMES
`dig <URL> -t <type>`

- [Zone transfer](https://digi.ninja/projects/zonetransferme.php) a domains DNS server
Zone transfers are used to copy a domain’s database from the primary server to the secondary server. If an attacker is able to perform a zone transfer with the primary or secondary name servers for a domain, the attacker will be able to view all DNS records for that domain.
`dig +short <URL>`
`dig axfr <target> @<nameserver>`

- Try doing zone transfer on int/internal keywords
`dig axfr <target (int/internal etc.)> @<nameserver>`

- Check bind version
`dig -t txt -c chaos VERSION.BIND @<TARGET>`

- Check for git repos
- Inspect Javascript files
