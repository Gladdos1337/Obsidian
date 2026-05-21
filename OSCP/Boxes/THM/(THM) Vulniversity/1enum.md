└─$ nmap -p- -T4 --min-rate 5000 10.114.160.45 -oN ports.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-21 10:25 -0400
Warning: 10.114.160.45 giving up on port because retransmission cap hit (6).
Nmap scan report for 10.114.160.45
Host is up (0.041s latency).
Not shown: 46079 closed tcp ports (reset), 19451 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3333/tcp open  dec-notes

Nmap done: 1 IP address (1 host up) scanned in 77.22 seconds
                                                                                                                    
┌──(kali㉿kali)-[~/Desktop/Kenobi]
└─$ nmap -p 21,22,139,445,3333 -sS -sC -A 10.114.160.45     
Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-21 10:31 -0400
Nmap scan report for 10.114.160.45
Host is up (0.034s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.5
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 f7:9f:7e:ea:e3:2c:3c:b4:1b:13:49:ef:7c:48:f9:0b (RSA)
|   256 a7:80:d5:c8:6c:13:e0:23:32:61:f2:75:7c:79:a5:31 (ECDSA)
|_  256 d6:c4:10:50:9d:ed:b9:b7:5d:23:a3:cf:f1:81:12:d9 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 4
445/tcp  open  netbios-ssn Samba smbd 4
3333/tcp open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Vuln University
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|phone
Running (JUST GUESSING): Linux 5.X|6.X|4.X (96%), Google Android 10.X|11.X|12.X (93%)
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:6 cpe:/o:linux:linux_kernel:4 cpe:/o:google:android:10 cpe:/o:google:android:11 cpe:/o:google:android:12 cpe:/o:linux:linux_kernel:5.4
Aggressive OS guesses: Linux 5.14 - 6.8 (96%), Linux 4.15 - 5.19 (96%), Linux 4.15 (96%), Linux 5.4 - 5.15 (96%), Android 10 - 12 (Linux 4.14 - 4.19) (93%), Android 10 - 11 (Linux 4.9 - 4.14) (92%), Android 12 (Linux 5.4) (92%), Android 9 - 11 (Linux 4.9 - 4.14) (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 3 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2026-05-21T14:32:38
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: , NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
