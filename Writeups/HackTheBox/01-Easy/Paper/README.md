```bash
nmap -sV -sC 10.129.136.31
```
```bash
Nmap scan report for office.paper (10.129.136.31)
Host is up (0.81s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey: 
|   2048 10:05:ea:50:56:a6:00:cb:1c:9c:93:df:5f:83:e0:64 (RSA)
|   256 58:8c:82:1c:c6:63:2a:83:87:5c:2f:2b:4f:4d:c3:79 (ECDSA)
|_  256 31:78:af:d1:3b:c4:2e:9d:60:4e:eb:5d:03:ec:a0:22 (ED25519)
80/tcp  open  http     Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
|_http-title: Blunder Tiffin Inc. &#8211; The best paper company in the elec...
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: WordPress 5.2.3
443/tcp open  ssl/http Apache httpd 2.4.37 ((centos) OpenSSL/1.1.1k mod_fcgid/2.3.9)
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=Unspecified/countryName=US
| Subject Alternative Name: DNS:localhost.localdomain
| Not valid before: 2021-07-03T08:52:34
|_Not valid after:  2022-07-08T10:32:34
|_ssl-date: TLS randomness does not represent time
|_http-title: HTTP Server Test Page powered by CentOS
|_http-server-header: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
|_http-generator: HTML Tidy for HTML5 for Linux version 5.7.28
| http-methods: 
|_  Potentially risky methods: TRACE

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.21 seconds
```
```bash
ffuf -c -w /usr/share/wordlists/dirb/big.txt -u https://10.129.136.31/FUZZ
```
```bash
________________________________________________

 :: Method           : GET
 :: URL              : https://10.129.136.31/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/big.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

.htpasswd               [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 324ms]
.htaccess               [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 570ms]
cgi-bin/                [Status: 403, Size: 199, Words: 14, Lines: 8, Duration: 525ms]
manual                  [Status: 301, Size: 237, Words: 14, Lines: 8, Duration: 322ms]
:: Progress: [20469/20469] :: Job [1/1] :: 80 req/sec :: Duration: [0:03:51] :: Errors: 0 ::
```

linpeas




```bash
find / -perm -4000 2>/dev/null
```
```bash
/usr/sbin/userhelper
/usr/sbin/mount.nfs
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/libexec/dbus-1/dbus-daemon-launch-helper
/usr/libexec/qemu-bridge-helper
/usr/libexec/cockpit-session
/usr/libexec/sssd/krb5_child
/usr/libexec/sssd/ldap_child
/usr/libexec/sssd/proxy_child
/usr/libexec/sssd/selinux_child
/usr/libexec/spice-gtk-x86_64/spice-client-glib-usb-acl-helper
/usr/libexec/Xorg.wrap
```

discovered service name 'polkit'
```bash
rpm -qa polkit
```
```
polkit-0.115-6.el8.x86_64
```
cve-2021-3560

