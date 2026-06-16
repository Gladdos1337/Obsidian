# Active Reconnaissance — CPTS Exam Reference

> **Prerequisite:** Complete [[01 - Information Gathering/1.1 Passive Recon/Passive Reconnaissance]] FIRST. Only touch the target when you have full passive picture.

---

## 1. Active Recon Methodology

### The Three-Phase Scan Approach

**Phase 1: Host Discovery** — Find live hosts
**Phase 2: Port Discovery** — Find ALL open ports (quick)
**Phase 3: Service Deep Scan** — Version + enumeration scripts on open ports

```
Phase 1:  sudo nmap -sn -T4 10.10.10.0/24             → live hosts
Phase 2:  sudo nmap -p- -T4 --min-rate 10000 $IP       → all open ports
Phase 3:  sudo nmap -sV -sC -p $PORTS $IP               → service details
```

### Why Phased Approach?
- Phase 1 filters dead hosts (massive time save on /24 or larger)
- Phase 2 is raw speed — no banner grabbing, just port detection
- Phase 3 is the expensive scan — only on open ports found

---

## 2. Host Discovery

### nmap Ping Sweep
```
# Standard ICMP echo + TCP 80 + TCP 443 + ICMP timestamp
sudo nmap -sn -T4 10.10.10.0/24

# ARP scan (local subnet only — fastest method)
sudo nmap -sn -PR 10.10.10.0/24

# Disable DNS resolution (faster)
sudo nmap -sn -n -T4 10.10.10.0/24

# Save output
sudo nmap -sn -T4 10.10.10.0/24 -oG hosts.gnmap
cat hosts.gnmap | awk '/Up/{print $2}' > live_hosts.txt
```

### netdiscover (ARP)
```
# Passive — listen for ARP
sudo netdiscover -r 10.10.10.0/24 -p

# Active — send ARP requests
sudo netdiscover -r 10.10.10.0/24

# Specific interface
sudo netdiscover -i eth0 -r 10.10.10.0/24

# Fast mode
sudo netdiscover -F -r 10.10.10.0/24
```

### arp-scan
```
# Quick ARP scan
sudo arp-scan --localnet
sudo arp-scan 10.10.10.0/24

# With interface
sudo arp-scan -I eth0 10.10.10.0/24

# Show MAC vendor (identify target OS)
sudo arp-scan --localnet | grep -i "pfsense\|vmware\|linux\|microsoft"
```

### masscan Host Discovery
```
# ICMP discovery
sudo masscan -p 80 10.10.10.0/24 --rate=1000

# Web only — faster than nmap for large ranges
sudo masscan -p 80,443,22,3389 10.10.10.0/24 --rate=10000 -oG web_hosts.gnmap
```

---

## 3. Masscan — The Speed King

### Basic Usage
```
# Scan specific ports
sudo masscan -p 80,443,22 10.10.10.10 --rate=1000

# Full port scan
sudo masscan -p1-65535 10.10.10.10 --rate=1000

# Subnet scan
sudo masscan -p80,443 10.10.10.0/24 --rate=10000

# Exclude IPs
sudo masscan -p1-65535 10.10.10.0/24 --rate=5000 --exclude 10.10.10.1

# Output grepable format
sudo masscan -p1-65535 10.10.10.10 --rate=1000 -oG masscan.gnmap
```

### Rate Tuning
```
# Slower — stable results
sudo masscan -p1-65535 10.10.10.10 --rate=500

# Fast — expect some packet loss
sudo masscan -p1-65535 10.10.10.10 --rate=5000

# LAN — very fast
sudo masscan -p1-65535 10.10.10.10 --rate=100000

# WAN — slower, be respectful
sudo masscan -p80,443,22,21 10.10.10.10 --rate=100
```

### Parsing Masscan Output
```
# Extract open ports
grep -oP '\d+(?=/open/tcp)' masscan.gnmap | sort -u | tr '\n' ',' | sed 's/,$//'

# Extract IP:Port
grep -oP '\d+\.\d+\.\d+\.\d+ \d+' masscan.gnmap | awk '{print $1 ":" $2}'
```

### Masscan → Nmap Pipeline
```
#!/bin/bash
IP=$1
sudo masscan -p1-65535 $IP --rate=1000 -oG masscan.gnmap
PORTS=$(grep -oP '\d+(?=/open/tcp)' masscan.gnmap | sort -u | tr '\n' ',' | sed 's/,$//')
sudo nmap -sV -sC -p$PORTS $IP -oN deep_scan.txt
```

---

## 4. Port Scanning Methodology

### Step 1: Quick All-Ports Scan
```
# Exam default — min-rate 10000, T4
sudo nmap -p- -T4 --min-rate 10000 --max-retries 1 $IP -oN quick_all.txt

# If packet loss detected, retry with
sudo nmap -p- -T4 --min-rate 5000 $IP -oN quick_all.txt

# For large subnets — masscan then nmap
sudo masscan -p1-65535 10.10.10.0/24 --rate=1000 -oG masscan.gnmap
```

### Step 2: Service Deep Scan on Open Ports
```
# Manual: extract ports, paste into next command
PORTS=21,22,80,139,443,445,3306,3389
sudo nmap -sV -sC -p$PORTS --version-intensity 7 $IP -oN service_scan.txt

# Automated from nmap output
PORTS=$(grep -oP '\d+(?=/tcp/open)' quick_all.txt | sort -u | tr '\n' ',' | sed 's/,$//')
sudo nmap -sV -sC -p$PORTS $IP -oN service_scan.txt
```

### Step 3: UDP Scan (Common Ports Only)
```
# Don't scan all 65535 UDP ports — pick the useful ones
sudo nmap -sU -p 53,67,68,69,111,123,135,137,138,139,161,162,500,514,520,1434,1900,4500 --max-retries 1 --min-rate 100 $IP -oN udp_scan.txt
```

### Step 4: Vulnerability Scan
```
# Targeted vuln scripts per service
sudo nmap -p 445 --script smb-vuln* $IP -oN smb_vulns.txt
sudo nmap -p 80,443 --script http-vuln* $IP -oN web_vulns.txt
sudo nmap -p 21 --script ftp-vsftpd-backdoor,ftp-proftpd-backdoor $IP -oN ftp_vulns.txt
```

---

## 5. Service-Specific Scanning Approaches

### Web Services (80, 443, 8080, 8443)
```
# Quick banners and headers
curl -s -I http://$IP | head -20
curl -s -k -I https://$IP | head -20

# Nmap deep web
sudo nmap -p 80,443 -sV --script http-enum,http-headers,http-title,http-server-header,http-methods,http-webdav-scan $IP
```

### SMB (139, 445)
```
# Always check NetBIOS first
nbtscan -r $IP/24

# SMB enumeration
smbclient -L //$IP -N                      # Null session
crackmapexec smb $IP --shares              # Share enumeration
sudo nmap -p 445 --script smb-enum-shares,smb-os-discovery $IP
```

### RDP (3389)
```
# Check if restricted admin
nmap -p 3389 --script rdp-sec-check $IP

# Credential check (be careful with rate limits)
crackmapexec rdp $IP -u admin -p password
```

### SSH (22)
```
# Authentication methods
nmap -p 22 --script ssh-auth-methods $IP

# Weak algorithms
nmap -p 22 --script ssh2-enum-algos $IP

# Direct connection test
ssh-keyscan -t rsa $IP
ssh root@$IP -o StrictHostKeyChecking=no
```

### FTP (21)
```
# Anonymous access
nmap -p 21 --script ftp-anon $IP
curl ftp://$IP --user anonymous:test

# Bounce scan
nmap -p 21 --script ftp-bounce $IP
```

### SQL (3306, 1433, 5432, 1521)
```
# MySQL
nmap -p 3306 --script mysql-empty-password,mysql-info,mysql-users $IP

# MSSQL
nmap -p 1433 --script ms-sql-info,ms-sql-empty-password $IP
```
	
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248```

```


# PostgreSQL
nmap -p 5432 --script pgsql-brute $IP

# Oracle
nmap -p 1521 --script oracle-sid-brute $IP
```

### SNMP (UDP 161)
```
# SNMP walk (v1/v2c — public community)
snmpwalk -v2c -c public $IP
snmpwalk -v1 -c public $IP 1.3.6.1.4.1.77.1.2.25    # Windows users
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.4.2.1.2    # Running processes
snmpwalk -v1 -c public $IP 1.3.6.1.2.1.25.1.6.0      # System processes
snmpwalk -v1 -c public $IP .1.3.6.1.2.1.1.5.0        # Hostname

# SNMP check
nmap -sU -p 161 --script snmp-info,snmp-processes,snmp-win32-software $IP
```

### NFS (2049)
```
# Show mounted exports
showmount -e $IP

# Mount directly
sudo mount -t nfs $IP:/exports /mnt/nfs -o nolock
```

---

## 6. Banner Grabbing

### Manual Netcat Banners
```
# Grab any TCP banner
nc -nv $IP 21                                    # FTP
nc -nv $IP 22                                    # SSH
nc -nv $IP 80 <<< "HEAD / HTTP/1.0"$'\r\n\r\n'  # HTTP
nc -nv $IP 443 <<< "HEAD / HTTP/1.0"$'\r\n\r\n'  # HTTPS (use openssl)
openssl s_client -connect $IP:443 -quiet <<< "HEAD / HTTP/1.0"$'\r\n\r\n'
nc -nv $IP 25                                    # SMTP
nc -nv $IP 110                                   # POP3
nc -nv $IP 143                                   # IMAP
nc -nv $IP 445                                   # SMB
```

### Scripted Banner Collection
```
#!/bin/bash
for port in 21 22 23 25 53 80 110 111 135 139 143 389 443 445 873 1433 1521 2049 3306 3389 5432 5900 5985 5986 6379 8080 8443 27017; do
  timeout 2 bash -c "echo > /dev/tcp/$1/$port" 2>/dev/null && echo "Port $port OPEN" || true
done
```

---

## 7. Port Knocking / Non-Standard Ports

Always scan all 65535 ports — services can live anywhere.
```
# Full scan — always do this
nmap -p- -T4 --min-rate 10000 $IP

# Then check all open ports manually
# Common services on weird ports:
#   - HTTP on 8080, 8443, 8000, 8888, 9000, 3000
#   - SSH on 2222, 22222, 222
#   - RDP on 3390, 3391, 3392
#   - MySQL on 3307, 3308
#   - PostgreSQL on 5433, 5434
```

---

## 8. Scanning Non-Target Subnets (Pivoting)

When you compromise one box and need to scan deeper:
```
# Through a SOCKS proxy (chisel, ssh -D, proxychains)
proxychains nmap -sT -p- -T4 172.16.1.0/24

# SSH tunnel forward scan
ssh -L 13389:10.10.10.10:3389 user@$JUMPBOX

# Chisel reverse SOCKS
# On target: ./chisel server -p 8080 --reverse
# On attack: ./chisel client $TARGET_IP:8080 R:1080:socks
# Then: proxychains nmap -sT -sV -p- internal.ip
```

---

## 9. Active Recon Workflow (Exam Checklist)

```
Phase 1: Host Discovery
  [] sudo nmap -sn -T4 10.10.10.0/24 -oG hosts.txt
  [] cat hosts.txt | awk '/Up/{print $2}' > live.txt

Phase 2: Port Discovery (per host)
  [] sudo nmap -p- -T4 --min-rate 10000 $IP -oN ports.txt
  [] Extract open ports list

Phase 3: Service Deep Scan
  [] sudo nmap -sV -sC -p $PORTS $IP -oN services.txt

Phase 4: UDP Scan
  [] sudo nmap -sU -p 53,161,123 --max-retries 1 $IP -oN udp.txt

Phase 5: Targeted Enumeration
  [] Run service-specific enumeration per open port
  [] Save ALL output for report evidence
```

---

## Related [[wikilinks]]
- [[01 - Information Gathering/Nmap Mastery]]
- [[01 - Information Gathering/1.1 Passive Recon/Passive Reconnaissance]]
- [[01 - Information Gathering/1.3 DNS Enumeration/DNS Enumeration]]
- [[01 - Information Gathering/1.4 Service Enumeration/Service Enumeration Compilation]]
