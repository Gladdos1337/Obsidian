Related: [[(THM) Skynet]] | [[LFI (Local File Inclusion)]] | [[RFI (Remote File Inclusion)]] | [[Types of shells]] | [[OSCP/Privilege Escalation/Shell/Tools]]

# Skynet - Resources & Further Reading

## Privilege Escalation (General)

These are the core resources — read these before attempting any privesc on a new box.

| Resource | What it covers |
|----------|---------------|
| [HackTricks - Linux Privesc](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html) | Comprehensive checklist — SUIDs, capabilities, cron, writable paths, sudo abuse, etc. |
| [GTFOBins](https://gtfobins.github.io/) | If a binary can be abused for privesc (sudo, SUID, capabilities) — look it up here first |
| [PayloadsAllTheThings - Linux Privesc](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Linux%20-%20Privilege%20Escalation.md) | Quick reference for every common technique with copy-paste commands |
| [THM - Linux PrivEsc room](https://tryhackme.com/room/linuxprivesc) | Hands-on practice room covering SUID, sudo, cron, writable /etc/passwd, NFS, etc. |
| [THM - Linux PrivEsc Arena](https://tryhackme.com/room/linuxprivescarena) | Practice room specifically built for drilling techniques |

---

## Cronjob / Wildcard Injection (used in this box)

| Resource | What it covers |
|----------|---------------|
| [HackTricks - Wildcards Spare tricks](https://book.hacktricks.wiki/en/linux-hardening/privilege-escalation/index.html#wildcards-spare-tricks) | Exactly the technique used here — tar wildcard injection explained |
| [Exploit-DB writeup on wildcard injection](https://www.exploit-db.com/papers/33930) | Original paper on Unix wildcard injection (tar, rsync, chown, chmod) |

**Key commands to remember for this technique:**
```bash
cat /etc/crontab              # check scheduled root jobs
cat /var/spool/cron/*         # per-user crontabs
ls -la /path/being/tarred/    # check if you can write to the target dir
```

---

## Automated Privesc Enumeration Scripts

Run these once you have a shell — they identify everything worth checking instantly.

```bash
# LinPEAS (most thorough)
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# LinEnum (lighter, faster)
./LinEnum.sh

# linux-exploit-suggester (kernel exploits)
./linux-exploit-suggester.sh
```

| Tool | Link |
|------|------|
| LinPEAS | https://github.com/peass-ng/PEASS-ng |
| LinEnum | https://github.com/rebootuser/LinEnum |
| linux-exploit-suggester | https://github.com/mzet-/linux-exploit-suggester |

---

## LFI / RFI (used in this box)

| Resource | What it covers |
|----------|---------------|
| [HackTricks - File Inclusion](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html) | LFI to RCE techniques, log poisoning, /proc tricks |
| [PayloadsAllTheThings - File Inclusion](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion) | Payloads and bypass techniques |

---

## SMB Enumeration

| Resource | What it covers |
|----------|---------------|
| [HackTricks - SMB](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smb/index.html) | Enumeration, null sessions, credential attacks |

---

## What to check on every Linux box (quick checklist)

```
sudo -l                          # what can you run as sudo?
find / -perm -4000 2>/dev/null   # SUID binaries
cat /etc/crontab                 # scheduled tasks
crontab -l                       # current user's crontab
env                              # environment variables (path hijacking?)
cat /etc/passwd                  # writable? any interesting users?
find / -writable -type f 2>/dev/null | grep -v proc   # writable files
uname -a                         # kernel version (for kernel exploits)
```
