#ctf #tryhackme
# Dogcat
---

## Nmap
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))

## Feroxbuster
/cats (403 Forbidden)
/dogs (403 Forbidden)

Inspect page source
Photos are named <number>.jpg in each directory

Running feroxbuster again to search for files with extensions jpg,txt,php
/cat.php
/dogs.php
/flag.php
/cats/<1-10>.jpg
/dogs/<1-10>.jpg

Try to change URL params

