# [HTB] MonitorsTwo - Penetration Testing Report
# Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.228.231
* **Vectors Used:** Cacti RCE (CVE-2022-46169) / Docker Privilege Escalation

# 1. Reconnaissance
Nmap was used for port scan against target
```bash
nmap -sV -sC 10.129.228.231
```
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-14 01:32 -0400
Nmap scan report for 10.129.228.231
Host is up (1.1s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Login to Cacti
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.76 seconds
```
<br>

Result:
* **22/SSH (OpenSSH 8.2p1)**
* **80/HTTP(nginx 1.18.0)**
<br>

# 2. Web Enumeration
There was no hidden directory or subdomains existed on the web page as a result of fuzzing. For more enumeration, hydra was also used for bruteforcing against web application, but no significant information was found.


