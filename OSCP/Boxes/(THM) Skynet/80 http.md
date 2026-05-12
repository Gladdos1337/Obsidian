80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST



gobuster dir -u 10.113.130.138 -w /usr/share/wordlists/dirb/common.txt