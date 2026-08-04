## What is Sshuttle

A Python tool that removes the need for **proxychains** entirely. Instead of prefixing every command with `proxychains`, sshuttle rewrites your local **iptables** rules so that traffic destined for a target subnet is transparently redirected through an SSH tunnel to the pivot host — no per-tool configuration needed.

**Limitation:** SSH-only. Unlike proxychains, it can't pivot over TOR or an HTTPS proxy.

---

## Sshuttle vs Proxychains

||Sshuttle|Proxychains|
|---|---|---|
|Transport|SSH only|SOCKS4/5, HTTP, TOR|
|Per-command config|None — works transparently at OS level|Must prefix every command: `proxychains nmap ...`|
|Mechanism|Rewrites iptables to redirect matching subnets|Injects into app's network calls via LD_PRELOAD|
|Setup|One command, one password prompt|Requires editing `/etc/proxychains.conf` + SSH `-D` tunnel first|
|UDP support|Off by default with `nat` method|Depends on proxy type|

---

## The Scenario

```
Attack Host ──SSH──> Ubuntu Pivot (10.129.202.64) ──> 172.16.5.0/23 (internal network)
```

Goal: scan/access hosts on `172.16.5.0/23` (e.g., `172.16.5.19`, a domain controller) as if they were directly reachable — without wrapping every command in proxychains.

---

## Full Attack Chain

### Step 1 — Install sshuttle (on attack host)

bash

```bash
sudo apt-get install sshuttle
```

### Step 2 — Run sshuttle, routing target subnet through the pivot

bash

```bash
sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v
```

- `-r ubuntu@10.129.202.64` — SSH remote/pivot host + credentials
- `172.16.5.0/23` — subnet to route through the pivot
- `-v` — verbose (shows the iptables rules being created live)

You'll be prompted for the SSH password. Once connected, sshuttle:

- Starts a firewall manager subprocess
- Sets up a TCP redirector on local port `12300`
- Injects `iptables`/`ip6tables` NAT rules so that any traffic destined for `172.16.5.0/32` gets transparently redirected into the SSH tunnel

### Step 3 — Use any tool directly, no proxychains needed

bash

```bash
sudo nmap -v -A -sT -p3389 172.16.5.19 -Pn
```

This just works — no `proxychains` prefix. The OS-level iptables rule intercepts the traffic and routes it through the SSH tunnel to the pivot before it ever reaches Nmap's raw socket layer.

---

## Why -sT is Required

Sshuttle relies on iptables NAT redirection, which only works cleanly with full TCP connections. That's why the Nmap scan uses `-sT` (full TCP connect scan) instead of `-sS` (SYN scan) — SYN scans send raw crafted packets that bypass the normal connect() flow sshuttle intercepts.

---

## Key Point

Sshuttle turns pivoting into an OS-level routing problem instead of an application-level proxy problem. Once it's running, your entire machine (for the specified subnet) behaves as if it's sitting on the other side of the pivot — any tool, any protocol over TCP, no extra configuration per tool. The tradeoff is it only speaks SSH to the pivot, so if you need TOR or HTTPS proxy chaining, you're back to proxychains.

---

## Tags

`#htb` `#cpts` `#pivoting` `#sshuttle` `#ssh` `#iptables` `#noProxychains` `#nmap`

![](/images/illustrations/session-progress.svg)![](/images/illustrations/session-progress-dark.svg)

See task progress for longer tasks.

SSH Pivoting with Sshuttle.md

SSH for Windows - Plink.md

Socat Redirection with a Bind Shell.md

![](/images/illustrations/session-context.svg)![](/images/illustrations/session-context-dark.svg)

Track tools and referenced files used in this task.