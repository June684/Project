# Executive Summary
* **Target OS:** Linux (Ubuntu / Docker Environment)
* **Difficulty:** Easy
* **Target IP:** 10.129.229.224
* **Vectors Used:** Metabase Pre-Auth RCE (CVE-2023-38646) & OverlayFS Privilege Escalation (CVE-2023-0386)

---

# 1. Reconnaisance
Port scan has been done via nmap against target machine.
```bash
nmap -sV -sC 10.129.229.224
```
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-20 04:29 -0400
Nmap scan report for 10.129.229.224
Host is up (0.77s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://analytical.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 35.95 seconds
```
from the result of scan above, there are three opened ports, which are potentially vulnerable for this system. Detailed information are shown below:
|port|status|service|
|:---|:---|:---|
|22|open|SSH(OpenSSH 8.2p1|
|80|open|http nginx 1.18.0 (Ubuntu)|

---

# 2. Web Enumeration
The webpage was unaccessable before configuring the accessibility via editting `/etc/hosts` file to redirect attacker's machine.
```bash
vi /etc/hosts
```
```bash
127.0.0.1    kali
127.0.0.1  localhost
10.129.229.224  analytical.htb (added for access the web application)
```
The web page was accessible after the change was made.

Similar situation happened when accessing the login page for further enumeration, which was located in subdomain `data.analytical.htb`. 
Same solution applied for encountered situation.
```bash
vi /etc/hosts
```
```bash
127.0.0.1    kali
127.0.0.1  localhost
10.129.229.224  analytical.htb  data.analytical.htb
```

<br>


# 3. Exploitation


# 4. Privilege Escalation


