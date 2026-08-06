**HTB Academy — Pivoting, Tunneling, and Port Forwarding (Section 11)**

---

## When to Use

- Compromised Windows host with access to internal network
- Windows host has 2 NICs (one external, one internal)
- Need to forward a specific port from attack host → internal service

**Example topology:**

```
Attack Host (10.10.15.5)
    ↓ port 8080
Pivot Host Windows 10 (10.129.15.150, 172.16.5.19)
    ↓ port 3389
Target (172.16.5.25 — internal only)
```

---

## Basic Syntax

cmd

```cmd
netsh.exe interface portproxy add v4tov4 listenport=<PORT> listenaddress=<PIVOT_IP> connectport=<TARGET_PORT> connectaddress=<TARGET_IP>
```

### Required Parameters

- `listenport` — port to open on pivot host
- `listenaddress` — pivot host's IP (usually the external-facing one)
- `connectport` — port on target service
- `connectaddress` — target host IP (internal network)

---

## Example — RDP Forwarding

Scenario: Compromise Windows 10 admin workstation, forward RDP to internal Windows Server.

cmd

```cmd
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```

**From attack host:**

bash

```bash
xfreerdp /v:10.129.15.150:8080 /u:victor /p:pass@123
```

---

## Verification

cmd

```cmd
C:\Windows\system32> netsh.exe interface portproxy show v4tov4
```

Output shows all active portproxy rules:

```
Listen on ipv4:             Connect to ipv4:
Address         Port        Address         Port
10.129.15.150   8080        172.16.5.25     3389
```

---

## Cleanup (Remove Rule)

cmd

```cmd
netsh.exe interface portproxy delete v4tov4 listenport=8080 listenaddress=10.129.15.150
```

---

## Limitations vs SSH Pivoting

|Feature|Netsh|SSH Dynamic (`-D`)|
|---|---|---|
|TCP forwarding|✓|✓|
|Specific port only|✓|✗ (full SOCKS)|
|UDP|✗|✗|
|Multiple ports|✓ (separate rules)|✓ (one tunnel)|
|Requires admin|✓|✗|

---

## Lab Workflow (HTB ACADEMY-PIVOTING-WIN10PIV)

cmd

```cmd
# 1. Gain shell on Windows pivot
# (via social engineering / phishing mentioned in HTB scenario)

# 2. Verify network interfaces
C:\> ipconfig /all
# Look for dual NICs (external + internal)

# 3. Add portproxy rule
C:\> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.25

# 4. Verify rule
C:\> netsh.exe interface portproxy show v4tov4

# 5. From attack host, connect through forward
bash$ xfreerdp /v:10.129.42.198:8080 /u:victor /p:pass@123
```

---

## Key Differences from SSH Pivoting

- **No SOCKS proxy** — each port needs its own rule
- **Admin required** — can't run as regular user
- **Static routes** — not flexible like dynamic tunnels
- **Good for:** Single service forwarding (RDP, SMB, SQL Server)
- **Bad for:** Network reconnaissance (no full tunnel)

---

## Notes

- Netsh rules persist across reboots (until explicitly deleted)
- Always verify `ipconfig /all` first to identify which IP is external-facing
- Use `Show v4tov4` to confirm rule before testing from attack host
- Port numbers don't need to match (can forward 8080→3389)

---

## Tags

`#htb` `#cpts` `#pivoting` `#windows` `#netsh` `#portforward`