## What is Plink

**Plink** (PuTTY Link) is a Windows command-line SSH client, bundled with the PuTTY package. Before late 2018, Windows had no native SSH client, so sysadmins relied on PuTTY/Plink. It supports the same tunneling modes as regular `ssh`: local forward, remote forward, and dynamic (SOCKS) forward.

**Why it matters for pivoting:** if you land on a Windows host that's locked down and you can't safely pull your own tools in, but PuTTY/Plink is already installed (or grabbable from a file share), you can live off the land and use it to pivot — no need to drop attacker tooling and risk detection.

Also relevant if your **attack host itself is Windows** — Plink + Proxifier replaces what SSH `-D` + proxychains would do on Linux.

---

## The Scenario

```
Windows Attack Host (10.10.15.5)
        │
        │  plink -ssh -D 9050 ubuntu@10.129.15.50
        ▼
   SSH session to Ubuntu pivot (10.129.15.50 / 172.16.5.129)
        │
        │  Ubuntu has a leg into 172.16.5.0/24
        ▼
   Windows A (172.16.5.19) — RDP service
```

Windows attack host cannot reach Windows A directly. Ubuntu can. Plink creates the encrypted tunnel; Proxifier routes RDP traffic (`mstsc.exe`) through it.

---

## Full Attack Chain

### Step 1 — Start Plink with dynamic port forwarding (SOCKS proxy)

cmd

```cmd
plink -ssh -D 9050 ubuntu@10.129.15.50
```

- `-ssh` — force SSH protocol
- `-D 9050` — dynamic port forward, opens a SOCKS listener on local port `9050`
- `ubuntu@10.129.15.50` — SSH login to the pivot host

This opens an SSH session Windows→Ubuntu and starts a SOCKS4 proxy on `127.0.0.1:9050` on the Windows attack host.

### Step 2 — Configure Proxifier

Windows native tools (like `mstsc.exe`) don't understand SOCKS proxies out of the box. **Proxifier** solves this — it intercepts traffic from chosen applications and forces it through a SOCKS/HTTPS proxy.

Proxifier profile:

- Address: `127.0.0.1`
- Port: `9050`
- Type: `SOCKS4`

### Step 3 — Launch mstsc.exe

With Proxifier configured and running, start `mstsc.exe` and RDP to the internal target (`172.16.5.19`). Proxifier transparently routes that RDP traffic through the SOCKS proxy → Plink → SSH tunnel → Ubuntu → internal network → Windows A.

---

## Traffic Flow Breakdown

```
mstsc.exe
   │  (RDP traffic, app has no SOCKS support)
   ▼
Proxifier (intercepts based on rules)
   │  (forwards into SOCKS proxy)
   ▼
127.0.0.1:9050  (SOCKS listener opened by Plink)
   │
   ▼
Plink SSH session → Ubuntu (10.129.15.50)
   │  (Ubuntu has interface on 172.16.5.129)
   ▼
172.16.5.19:3389  (Windows A RDP service)
```

---

## Key Concepts

|Component|Role|
|---|---|
|Plink|Windows CLI SSH client — builds the encrypted tunnel + SOCKS listener|
|`-D 9050`|Dynamic port forward = SOCKS proxy on local port 9050|
|Proxifier|Forces non-SOCKS-aware Windows apps (mstsc, etc.) through the SOCKS proxy|
|Ubuntu (pivot)|Dual-homed host relaying SOCKS traffic into the internal network|

---

## Plink vs Linux SSH

||Plink (Windows)|ssh (Linux)|
|---|---|---|
|Dynamic forward|`plink -ssh -D 9050 user@host`|`ssh -D 9050 user@host`|
|Local forward|`plink -L local:remote:port user@host`|`ssh -L local:remote:port user@host`|
|Remote forward|`plink -R remote:local:port user@host`|`ssh -R remote:local:port user@host`|
|SOCKS-unaware app support|Needs Proxifier|Needs proxychains|

---

## Key Point

Plink does exactly what SSH does for tunneling, just packaged for Windows. The extra moving part on Windows is **Proxifier** — it's the bridge that lets GUI apps without built-in SOCKS support (like `mstsc.exe`) still ride through the SOCKS proxy Plink creates. Two independent mechanisms (SOCKS dynamic forward + a separate remote forward of 8080→80 shown in the diagram) can run simultaneously over the same Plink SSH session.

---

## Tags

`#htb` `#cpts` `#pivoting` `#plink` `#putty` `#proxifier` `#socks` `#windowsAttackHost` `#dynamicPortForward`