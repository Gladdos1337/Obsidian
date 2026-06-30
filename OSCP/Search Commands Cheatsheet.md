# Search Commands Cheatsheet

---

## Finding Flags

```bash
# Find any file named flag or user/root txt
find / -name "flag*" 2>/dev/null
find / -name "user.txt" 2>/dev/null
find / -name "root.txt" 2>/dev/null

# Search by content if you don't know the filename
grep -r "THM{" / 2>/dev/null
# Search for any directory in root that isn't standard
find / -maxdepth 1 -not -path "/" -not -path "/proc" -not -path "/sys" -not -path "/dev" 2>/dev/null
# Search for recently modified by admins (shit):
find / -type f -user root -mtime -30 2>/dev/null

```

---

## Finding Specific thing

```bash
# Find file in a specific folder | display line that have "api" in it 
find /opt/SecLists/ | grep -i api

#Find how much characters there is in the string:
echo -n ASDASFKAPGFKEPOKEPO3412fQWFKQOW | wc -c



```

---

## Finding Files by Owner

```bash
# Files owned by root but writable by others
find / -user root -perm -o+w 2>/dev/null

# Files owned by current user
find / -user $(whoami) 2>/dev/null

# Files owned by a specific user
find / -user milesdyson 2>/dev/null
```

---

## Finding Credentials

```bash
# Config files that often contain passwords
find / -name "*.conf" 2>/dev/null
find / -name "*.config" 2>/dev/null
find / -name "wp-config.php" 2>/dev/null
find / -name "config.php" 2>/dev/null
find / -name ".env" 2>/dev/null

# Search file contents for passwords
grep -r "password" /etc 2>/dev/null
grep -r "passwd" /var/www 2>/dev/null
grep -ri "password" /home 2>/dev/null

# SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "authorized_keys" 2>/dev/null
find / -name "*.pem" 2>/dev/null
```

---

## Finding Recently Modified Files

```bash
# Modified in last 10 minutes
find / -mmin -10 2>/dev/null

# Modified in last 24 hours
find / -mtime -1 2>/dev/null

# Modified in last 7 days, exclude /proc and /sys
find / -mtime -7 ! -path "/proc/*" ! -path "/sys/*" 2>/dev/null
```

---

## Finding Interesting Files

```bash
# Bash history (credentials often here)
cat ~/.bash_history
cat /home/*/.bash_history

# Crontabs
cat /etc/crontab
ls -la /etc/cron*
crontab -l

# Readable /etc/shadow (misconfigured boxes)
cat /etc/shadow

# Writable /etc/passwd (can add root user)
ls -la /etc/passwd

# Backup files
find / -name "*.bak" 2>/dev/null
find / -name "*.backup" 2>/dev/null
find / -name "*.old" 2>/dev/null
find / -name "*.zip" 2>/dev/null
find / -name "*.tar.gz" 2>/dev/null

# Database files
find / -name "*.db" 2>/dev/null
find / -name "*.sqlite" 2>/dev/null
```

---

## Finding Running Processes and Services

```bash
# Running processes
ps aux

# Open network connections
netstat -tulpn
ss -tulpn

# Services listening only on localhost (internal ports)
ss -tnlp | grep 127.0.0.1
```

---

## Searching Inside Files

```bash
# Search for a string in all files in current directory
grep -r "search term" .

# Case insensitive
grep -ri "password" .

# Show filename and line number
grep -rn "password" /var/www

# Search multiple extensions
grep -r "password" --include="*.php" /var/www
grep -r "password" --include="*.conf" /etc
```

---

## Quick Wins to Always Check

```bash
# Can you read /etc/shadow?
cat /etc/shadow

# Is /etc/passwd writable?
ls -la /etc/passwd

# Any credentials in environment variables?
env
printenv

# What's in root's home? (sometimes readable)
ls -la /root

# NFS shares with no_root_squash?
cat /etc/exports

# Any capabilities set on binaries?
getcap -r / 2>/dev/null
```

---

## Filtering Out Noise

Always append `2>/dev/null` to suppress "Permission denied" errors.

```bash
# Without filter — unusable output
find / -name "*.php"

# With filter — clean output
find / -name "*.php" 2>/dev/null
```
