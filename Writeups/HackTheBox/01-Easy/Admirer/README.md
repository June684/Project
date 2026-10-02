# [HTB] Admirer - Penetration Testing Report

## Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.229.101
* **Vectors Used:** Adminer 4.7.8 LFI/SSR Vulnerability & Python Library Hijacking

---

## 1.  Reconnaissance

Port scan has been done via nmap against target.
```bash
$ nmap -sV -sC -p- 10.129.229.101 -oN
```
result:
```bash
NSE Timing: About 97.90% done; ETC: 01:11 (0:00:00 remaining)
Nmap scan report for 10.129.229.101
Host is up (0.76s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a:71:e9:21:63:69:9d:cb:dd:84:02:1a:23:97:e1:b9 (RSA)
|   256 c5:95:b6:21:4d:46:a4:25:55:7a:87:3e:19:a8:e7:02 (ECDSA)
|_  256 d0:2d:dd:d0:5c:42:f8:7b:31:5a:be:57:c4:a9:a7:56 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-title: Admirer
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 45.93 seconds
```

from the result of scan above, there are three opened ports, which are potentially vulnerable for this system.
Detailed information are shown below:
|port|status|service|
|:---|:---|:---|
|21|open|FTP(vsftpd 3.0.3)|
|22|open|SSH(OpenSSH 7.4p1|
|80|open|HTTP(Apache httpd 2.4.25|

Based on the given information, enumeration was initiated to opened port 21. Anonymous logon method was initated, however,
permission was denied to access.

Since there were no information given about SSH logon credentials, web app enumeration was initiated for further reconnaissance.

[This](./images/AdmirerWebApp.png) is the web page running on the target system.
