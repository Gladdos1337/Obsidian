PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Annoucement
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 5.14 - 6.8 (95%), Linux 4.15 - 5.19 (95%), Linux 4.15 (94%), Linux 3.11 - 3.14 (92%), Linux 3.7 - 4.19 (92%), Ruckus ZoneFlex R710 WAP (Linux 3.4) (92%), Linux 3.10 (92%), Linux 3.2 - 4.14 (92%), Linux 3.8 - 3.16 (92%)
No exact OS matches for host (test conditions non-ideal).
Uptime guess: 28.209 days (since Wed Apr 15 03:57:15 2026)
Network Distance: 3 hops
TCP Sequence Prediction: Difficulty=258 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT      ADDRESS
1   37.69 ms 192.168.128.1
2   ...
3   36.14 ms 10.112.165.59

NSE: Script Post-scanning.
Initiating NSE at 08:58
Completed NSE at 08:58, 0.00s elapsed
Initiating NSE at 08:58
Completed NSE at 08:58, 0.00s elapsed
Initiating NSE at 08:58
Completed NSE at 08:58, 0.00s elapsed
Read data files from: /usr/share/nmap
OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.75 seconds
           Raw packets sent: 65855 (2.900MB) | Rcvd: 65580 (2.625MB)