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

Successfully gained access to login page via visiting url `data.analytical.htb`.

<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/fe2908ed-e520-4101-bb10-104097ed5cb9" />


# 3. Exploitation
## 1. CVE-2023-38646 
Confirmed login page was running by `Metabase` instance, checking the version was necessary for proper exploitation, which was `v0.46.6`.
```bash
curl http://data.analytical.htb | grep version
```
It is confirmed that the version of `Metabase` is vulnerable to `CVE-2023-38646 (Pre-Authentication Remote Code Execution)`, which is occurred because of the accessibility of users accessing a special token called `setup-token` used for completing the setup process. Unauthorized users, as well, could access the `setup-token` even after the setup process is done, since it remained active.


# 4. Privilege Escalation


