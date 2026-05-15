Related: [[Passive v Active]] | [[Brute force]] | [[Types of shells]]

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

nmap -p 21,22,80 -sV -sC -A 10.114.149.14-oN detailed.txt

| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.232.84
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 89:33:53:1b:9a:1e:a4:ba:7a:64:4a:fa:26:02:c7:90 (RSA)
|   256 d9:af:6f:13:55:07:cd:47:ef:25:14:a0:48:dc:8d:27 (ECDSA)
|_  256 bf:e6:6c:ae:29:ca:2e:c2:f6:40:66:66:77:53:43:be (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)



ssh lin@10.114.191.131
RedDr4gonSynd1cat3
RedDr4gonSynd1cat3

THM{80UN7Y_h4cK3r}
