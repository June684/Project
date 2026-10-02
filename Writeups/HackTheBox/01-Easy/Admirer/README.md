# [HTB] Admirer - Penetration Testing Report

## Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.229.101
* **Vectors Used:** Adminer 4.7.8 LFI/SSR Vulnerability & Python Library Hijacking

---

## 1. Reconnaissance

Port scan has been done via nmap against target.
```bash
$ nmap -sV -sC -p- 10.129.229.101 -oN
```
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
<br>

from the result of scan above, there are three opened ports, which are potentially vulnerable for this system.
Detailed information are shown below:
|port|status|service|
|:---|:---|:---|
|21|open|FTP(vsftpd 3.0.3)|
|22|open|SSH(OpenSSH 7.4p1|
|80|open|HTTP(Apache httpd 2.4.25|
<br>
Based on the given information, enumeration was initiated to opened port 21. Anonymous logon method was initated, however,
permission was denied to access.

Since there were no information given about SSH logon credentials, web app enumeration was initiated for further reconnaissance.  
[This](./images/AdmirerWebApp.png) is the web page running on the target system.


## 2. Web Enumeration

Since there was no revealed information about web service, web content discovery was inevitable. For the task, `gobuster` and `ffuf` was used, specifically looking for any .php or .txt files included on the web service.
```bash
gobuster dir -u http://10.129.229.101/ -w /usr/share/seclists/Discovery/Web-Content/common.txt -x php,txt
```
```bash
===============================================================
[+] Url:                     http://10.129.229.101
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.hta                 (Status: 403) [Size: 279]
.htaccess            (Status: 403) [Size: 279]
.hta.txt             (Status: 403) [Size: 279]
.hta.php             (Status: 403) [Size: 279]
.htpasswd.txt        (Status: 403) [Size: 279]
.htaccess.txt        (Status: 403) [Size: 279]
.htaccess.php        (Status: 403) [Size: 279]
.htpasswd.php        (Status: 403) [Size: 279]
.htpasswd            (Status: 403) [Size: 279]
assets               (Status: 301) [Size: 317] [--> http://10.129.229.101/assets/]
```
<br>
<br>

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.129.229.101/FUZZ -o .php,.txt
```
```bash
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.229.101/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Output file      : .txt,.php
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 308ms]
.hta                    [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 308ms]
.htaccess               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 305ms]
assets                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 304ms]
images                  [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 304ms]
index.php               [Status: 200, Size: 6051, Words: 385, Lines: 154, Duration: 409ms]
robots.txt              [Status: 200, Size: 138, Words: 21, Lines: 5, Duration: 561ms]
server-status           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 286ms]
:: Progress: [4750/4750] :: Job [1/1] :: 97 req/sec :: Duration: [0:00:44] :: Errors: 0 ::
```
<br>

Based on the scan, we discovered `robots.txt`, `index.php` on this web service. Since the status of both files is not `forbidden(403)`, it is `allowed(200)` to look into the contents of the discovered files. It is discovered that `index.php` redirects to the main page instead of showing its content, the only option left is to check the content of `robots.txt`.
```bash
http://10.129.229.101/robots.txt
```


```bash
User-agent: *

# This folder contains personal contacts and creds, so no one -not even robots- should see it - waldo
Disallow: /admin-dir


