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
|:---|:---|:---|
|21|open|FTP(vsftpd)|
|22|open|SSH(OpenSSH 8.4p1)|
|80|open|HTTP(nginx 1.18.0)|
<br>

# 2. Vulnerability Assessment
Following is the main page of target machine, which was powered by `Wordpress`



