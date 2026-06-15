# Nmap Mastery — CPTS Exam Reference

> **Core philosophy:** Always run the quick scan first, then deep scan on discovered ports. Never waste time scanning all 65535 ports with service scripts on the first pass.

---

## 1. Scan Types — When to Use Each

### SYN Scan (`-sS`) — Default & Stealth
```
sudo nmap -sS -p- -T4 10.10.10.10
```
- **Half-open**: sends SYN, RST on SYN-ACK. Never completes handshake.
- **Fastest** scan type. Requires root. Default if running as root.
- Use for: initial discovery, stealth against non-Windows hosts.
- **Note:** Windows hosts often don't respond to SYN scans on closed ports cleanly — consider `-sT` instead.

### TCP Connect Scan (`-sT`) — No Root Required
```
nmap -sT -p- -T4 10.10.10.10
```
- Completes full TCP handshake. Slower, logged by apps.
- Use when: not root, or scanning Windows hosts where SYN scan is unreliable.
- Default when running as non-root.

### UDP Scan (`-sU`) — Slow But Necessary
```
sudo nmap -sU -p 53,67-69,111,123,135,137-139,161-162,445,500,514,520,631,1434,1900,4500 10.10.10.10
```
- **Very slow.** ICMP port-unreachable = closed. No response = open|filtered.
- Always use `--max-retries 1` and `--min-rate 100` to speed up.
- Target common UDP ports only unless you have all day.

### FIN Scan (`-sF`) — Stealth Through Stateful Firewalls
```
sudo nmap -sF -p 80,443,22 10.10.10.10
```
- Sends FIN flag. RFC-compliant hosts send RST on closed, ignore on open.
- Bypasses some non-stateful firewalls. MS Windows ignores all FIN packets — useless there.

### Xmas Scan (`-sX`) & Null Scan (`-sN`)
```
sudo nmap -sX -p 80,443,22 10.10.10.10   # FIN+PSH+URG flags
sudo nmap -sN -p 80,443,22 10.10.10.10   # No flags
```
- Same concept as FIN. Bypasses stateless packet filters.
- **Not for Windows** — they ignore regardless.

### ACK Scan (`-sA`) — Map Firewall Rules
```
sudo nmap -sA -p 80,443,22 10.10.10.10
```
- Sends ACK. Unfiltered → RST. No response → filtered.
- **Does not discover open ports** — discovers which ports the firewall allows through.
- Combine with `-sW` (Window scan) to sometimes determine open vs filtered.

### Window Scan (`-sW`)
```
sudo nmap -sW -p 80,443,22 10.10.10.10
```
- Like ACK scan but examines TCP Window field. Some systems return non-zero window on open ports.
- Rarely useful on modern systems.

---

## 2. NSE Scripts — Organized by Category

### Discovery & Enumeration (Run First)
```
# Service version + default scripts
sudo nmap -sV -sC -p 80,443,22,445 10.10.10.10

# Full safe scripts
sudo nmap -sV --script safe -p- 10.10.10.10
```

### Vulnerability Scanning
```
sudo nmap --script vuln -p 80,443,445 10.10.10.10
```
> **Warning:** `vuln` includes intrusive scripts that can crash services. Prefer targeted scripts.

### Brute Force
```
sudo nmap --script brute -p 22,3389,445 10.10.10.10
```
- Contains: ssh-brute, rdp-brute, smb-brute, http-brute, ftp-brute, mysql-brute
- **Slow** — limit to 1-2 default credentials first.

### Auth Bypass
```
sudo nmap --script auth -p 80,443,445 10.10.10.10
```
- Tests for anonymous FTP, SMB null sessions, default credentials.

### Malware Detection
```
sudo nmap --script malware -p 80,443 10.10.10.10
```

### DOS Testing (Be Careful)
```
sudo nmap --script dos --script-args vulns.showall -p 80 10.10.10.10
```

### Broadcast Discovery (LAN)
```
sudo nmap --script broadcast-dns-service-discovery
sudo nmap --script broadcast-ping --script-args broadcast-ping.foreground
```

### Key Individual Scripts to Know

| Script | Purpose | Port |
|--------|---------|------|
| `http-enum` | Directory/file brute forcing | 80,443 |
| `http-webdav-scan` | WebDAV misconfigs | 80,443 |
| `http-shellshock` | Shellshock test | 80,443 |
| `smb-vuln-ms17-010` | EternalBlue | 445 |
| `smb-vuln-ms08-067` | Conficker | 445 |
| `smb2-security-mode` | SMB signing check | 445 |
| `ssl-heartbleed` | Heartbleed | 443 |
| `ssl-enum-ciphers` | Weak SSL ciphers | 443 |
| `ftp-anon` | Anonymous FTP | 21 |
| `mysql-empty-password` | Root no-pass | 3306 |
| `rdp-sec-check` | RDP security | 3389 |
| `dns-zone-transfer` | Zone transfer | 53 |
| `dns-brute` | Subdomain brute | 53 |

---

## 3. Performance Tuning

### Timing Templates (`-T0` to `-T5`)

| Template | Name | When to Use |
|----------|------|-------------|
| `-T0` | Paranoid | IDS evasion, serial scans |
| `-T1` | Sneaky | IDS evasion, 1 probe every 15s |
| `-T2` | Polite | Slower than default, less bandwidth |
| `-T3` | Normal | Default — good for most |
| `-T4` | Aggressive | **Exam default** — reliable LAN |
| `-T5` | Insane | CAPTURE THE FLAG / lab only |

**Exam Recommendation:** Start with `-T4`. Drop to `-T3` if packet loss or timeouts.

### Fine-Tuning Parameters

```
# Minimum rate — force scan speed
sudo nmap -p- --min-rate 10000 10.10.10.10     # 10k pkts/sec
sudo nmap -p- --min-rate 50000 10.10.10.10     # Aggressive (CTF)

# Parallelism — concurrent probes
sudo nmap -p- --min-parallelism 100 10.10.10.10 # Always 100 probes

# Retries — reduce for faster scans, increase for reliability
sudo nmap -p- --max-retries 1 10.10.10.10       # Faster, less reliable
sudo nmap -p- --max-retries 3 10.10.10.10       # Default

# Host timeout — give up on slow hosts
sudo nmap -p- --host-timeout 30m 10.10.10.10
```

### Batch Scan Recipe (Fast Discovery)
```
# Phase 1: Quick discovery (all ports, no service scan)
sudo nmap -p- -T4 --min-rate 10000 --max-retries 1 10.10.10.0/24 -oN discovery.txt

# Phase 2: Service scan on open ports
PORTS=$(grep -oP '\d+(?=/tcp)' discovery.txt | tr '\n' ',' | sed 's/,$//')
sudo nmap -sV -sC -p$PORTS 10.10.10.10 -oN services.txt
```

---

## 4. Firewall/IDS Evasion

### Packet Fragmentation
```
sudo nmap -f -p 80,443 10.10.10.10         # 8-byte fragments
sudo nmap -f -f -p 80,443 10.10.10.10      # 16-byte fragments
sudo nmap --mtu 24 -p 80,443 10.10.10.10   # Custom MTU (must be 8x)
```

### Decoy Scanning
```
sudo nmap -D RND:10 -p 80,443 10.10.10.10    # 10 random decoys
sudo nmap -D 10.10.10.1,10.10.10.2,ME -p 80,443 10.10.10.10  # Specific decoys + you
```

### Source Port Manipulation
```
sudo nmap --source-port 53 -p 80,443 10.10.10.10    # DNS-port bypass
sudo nmap --source-port 20 -p 80,443 10.10.10.10    # FTP-data bypass

# If you get a shell, use the same technique:
socat -T10 TCP-LISTEN:12345,fork,reuseaddr TCP:10.10.10.10:80,bind=10.10.10.10,sourceport=53
```

### Data Length Padding
```
sudo nmap --data-length 500 -p 80,443 10.10.10.10   # Appends random data
```

### TTL Manipulation
```
sudo nmap --ttl 128 -p 80,443 10.10.10.10
```

### MAC Spoofing
```
sudo nmap --spoof-mac 00:11:22:33:44:55 -p 80,443 10.10.10.10
sudo nmap --spoof-mac Cisco -p 80,443 10.10.10.10         # Vendor OUI
sudo nmap --spoof-mac 0 -p 80,443 10.10.10.10             # Random MAC
```

### Idle Scan (Zombie)
```
# Find an idle zombie host first
sudo nmap -p 80 -sI 10.10.10.5 10.10.10.10
```
- Nearly useless on modern networks with traffic. Know it exists for exam theory.

---

## 5. Output Parsing

### Output Formats
```
sudo nmap -oN scan.txt          # Normal — human readable
sudo nmap -oG scan.gnmap        # Grepable — parse with grep/awk
sudo nmap -oX scan.xml          # XML — for xsltproc
sudo nmap -oA scan              # ALL formats (prefix = "scan")
```

### Parsing Grepable Output
```
# Extract open ports
grep -oP '\d+(?=/open/tcp)' scan.gnmap | sort -u
grep -oP '\d+/open' scan.gnmap | cut -d/ -f1

# Extract IPs with specific ports open
grep ',445/open/' scan.gnmap | awk '{print $2}'
```

### XML to HTML Report
```
# Install xsltproc first
sudo apt install xsltproc -y

# Convert
xsltproc -o scan.html scan.xml /usr/share/nmap/nmap.xsl
```

### Masscan → Nmap Pipeline
```
# Discovery with masscan
sudo masscan -p1-65535 --rate=1000 10.10.10.10 -oG masscan.gnmap

# Parse masscan grepable output
PORTS=$(awk '/open/{print $4}' masscan.gnmap | cut -d/ -f1 | sort -u | tr '\n' ',')

# Feed to nmap for deep scan
sudo nmap -sV -sC -p$PORTS 10.10.10.10 -oN deep_scan.txt
```

---

## 6. Common Scan Recipes

### Quick Scan (Always First)
```
sudo nmap -p- -T4 --min-rate 10000 --max-retries 1 -oN quick_scan.txt $IP
```

### Targeted Deep Scan
```
sudo nmap -sV -sC -p 21,22,25,53,80,110,135,139,143,389,443,445,873,1433,1521,2049,3306,3389,5432,5985,5986,6379,27017 -oN service_scan.txt $IP
```

### Stealth Scan
```
sudo nmap -sS -T2 -f --source-port 53 --data-length 200 -D RND:5 -p 80,443,22,445 $IP
```

### Full Vulnerability Scan
```
sudo nmap --script vuln --script-args vulns.showall -p 21,22,25,80,443,445,3306,3389,5432,6379 -oN vuln_scan.txt $IP
```

### SMB Deep Scan
```
sudo nmap -p 445 --script smb-vuln*,smb-enum*,smb-os-discovery,smb-security-mode -oN smb_scan.txt $IP
```

### Web Deep Scan
```
sudo nmap -p 80,443 --script http-enum,http-webdav-scan,http-shellshock,http-headers,http-title,http-server-header -oN web_scan.txt $IP
```

### Host Discovery (Subnet)
```
sudo nmap -sn -T4 --min-rate 1000 10.10.10.0/24 -oN hosts.txt
```

### Top Ports Scan (Fast)
```
sudo nmap --top-ports 1000 -sV -T4 $IP
sudo nmap --top-ports 100 --sV -T3 $IP
```

---

## 7. Service-Specific Scripts

### SMB (445)
```
sudo nmap -p 445 --script smb-vuln-ms17-010,smb-vuln-ms08-067,smb-enum-shares,smb-os-discovery,smb-security-mode,smb2-security-mode,smb-enum-users $IP
```

### HTTP/HTTPS (80,443)
```
sudo nmap -p 80,443 --script http-enum,http-headers,http-title,http-server-header,http-webdav-scan,http-methods,http-shellshock,http-php-version,http-apache-negotiation $IP
```

### FTP (21)
```
sudo nmap -p 21 --script ftp-anon,ftp-bounce,ftp-vsftpd-backdoor,ftp-proftpd-backdoor $IP
```

### SSH (22)
```
sudo nmap -p 22 --script ssh-auth-methods,ssh2-enum-algos,ssh-hostkey $IP
```

### MySQL (3306)
```
sudo nmap -p 3306 --script mysql-empty-password,mysql-users,mysql-info,mysql-audit,mysql-databases,mysql-variables $IP
```

### MSSQL (1433)
```
sudo nmap -p 1433 --script ms-sql-info,ms-sql-ntlm-info,ms-sql-empty-password,ms-sql-brute $IP
```

### RDP (3389)
```
sudo nmap -p 3389 --script rdp-sec-check,rdp-enum-encryption $IP
```

### SMTP (25)
```
sudo nmap -p 25 --script smtp-commands,smtp-open-relay,smtp-enum-users $IP
```

### DNS (53)
```
sudo nmap -p 53 --script dns-zone-transfer,dns-brute,dns-srv-enum,dns-cache-snoop $IP
```

### SNMP (161)
```
sudo nmap -sU -p 161 --script snmp-brute,snmp-info,snmp-processes,snmp-win32-software,snmp-netstat $IP
```

### NFS (2049)
```
sudo nmap -p 2049 --script nfs-showmount,nfs-ls,nfs-statfs $IP
```

---

## 8. UDP Scanning

### Why UDP scanning matters
- Services like SNMP, DNS, DHCP, TFTP, NTP, NetBIOS run over UDP
- Often overlooked = quick win in exams
- Slow because stateless protocol — nmap sends probe, waits for response or ICMP unreachable

### Optimized UDP Scan
```
# Fast UDP scan — only common ports
sudo nmap -sU -p 53,67,68,69,111,123,135,137,138,139,161,162,500,514,520,1434,1900,4500 --max-retries 1 --min-rate 100 -T4 $IP

# Slightly deeper
sudo nmap -sU -p 53,67-69,111,123,135,137-139,161-162,445,500,514,520,631,1434,1900,4500,5353 -T4 $IP
```

### Service-Specific UDP Checks
```
# SNMP brute
sudo nmap -sU -p 161 --script snmp-brute --script-args snmp-brute.communitiesdb=/usr/share/seclists/Discovery/SNMP/common-snmp-communities-onesixtyone.txt $IP

# NTP monlist
sudo nmap -sU -p 123 --script ntp-monlist $IP

# DNS cache snooping
sudo nmap -sU -p 53 --script dns-cache-snoop $IP

# DHCP discovery
sudo nmap -sU -p 67,68 --script broadcast-dhcp-discover
```

### onesixtyone (Faster UDP SNMP Scan)
```
# Install
sudo apt install onesixtyone -y

# Use
echo public > community.txt
echo private >> community.txt
echo manager >> community.txt
onesixtyone -c community.txt -i hosts.txt
```

---

## Quick Reference Card

```
SCAN TYPE          FLAG       SPEED    ROOT?    EVASION
SYN                -sS        ***      Yes      Partial
TCP Connect        -sT        **       No       None
UDP                -sU        *        Yes      None
FIN                -sF        ***      Yes      Some
Xmas               -sX        ***      Yes      Some
Null               -sN        ***      Yes      Some
ACK                -sA        ***      Yes      Firewall map
Window             -sW        ***      Yes      Rarely useful

PERFORMANCE:
Default timing:    -T3
Fast (exam):       -T4 --min-rate 10000 --max-retries 1
Stealth:           -T1 or -T2 -f --source-port 53 -D RND:5

OUTPUT:
Normal:            -oN
Grepable:          -oG
XML:               -oX
All:               -oA

ALWAYS REMEMBER:
1. Quick all-ports scan first (-p-)
2. Service version + scripts on found ports (-sV -sC)
3. UDP scan only common ports (-sU -p 53,161,...)
4. Save output every time (-oA)
5. Parse open ports efficiently with grep -oP
```

---

## Related [[wikilinks]]
- [[01 - Information Gathering/1.2 Active Recon/Active Reconnaissance]]
- [[01 - Information Gathering/1.4 Service Enumeration/Service Enumeration Compilation]]
- [[Passive Reconnaissance]]
- [[DNS Enumeration]]
