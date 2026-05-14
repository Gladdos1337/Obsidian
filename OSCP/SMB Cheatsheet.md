# SMB Cheatsheet

---

## Enumeration

```bash
# Nmap SMB scripts
nmap -p 445 --script smb-enum-shares,smb-enum-users <ip>
nmap -p 445 --script smb-vuln* <ip>           # check for known vulns (EternalBlue etc.)

# List shares — no creds (null session)
smbclient -L //<ip> -N

# List shares — with creds
smbclient -L //<ip> -U <user>

# enum4linux — full SMB recon (users, shares, groups, OS)
enum4linux -a <ip>

# crackmapexec — quick share listing
crackmapexec smb <ip> --shares
crackmapexec smb <ip> --shares -u '' -p ''    # null session
crackmapexec smb <ip> --shares -u <user> -p <password>
```

---

## Connecting to a Share

```bash
# Anonymous / null session
smbclient //<ip>/<share> -N

# With credentials
smbclient //<ip>/<share> -U <user>
smbclient //<ip>/<share> -U <user>%<password>   # inline password
```

---

## Navigating Inside smbclient

```bash
ls                        # list files
cd <dir>                  # change remote directory
lcd <dir>                 # change local directory (where downloads land)
pwd                       # show remote working directory
```

---

## Downloading Files

```bash
get <filename>            # download a single file
get <filename> <local>    # download and rename locally

mget *                    # download all files in current directory
mget *.txt                # wildcard — only matching files

prompt                    # toggle per-file confirmation off (run before mget)
recurse                   # toggle recursive mode on (for nested folders)
```

> Run `prompt` and `recurse` before `mget *` to download everything non-interactively.

---

## Uploading Files

```bash
put <local_file>                      # upload to current remote directory
put <local_file> <remote_filename>    # upload with a different name

mput *                                # upload all files from local directory
```

---

## Reading a File Without Downloading

```bash
# Pipe the file to stdout (read directly in terminal)
smbclient //<ip>/<share> -N -c 'get <filename> /dev/stdout'
```

---

## Download Everything Recursively (one-liner)

```bash
# Using smbclient with mask + recurse
smbclient //<ip>/<share> -N -c 'prompt; recurse; mget *'

# Using smbget (simpler for recursive download)
smbget -R smb://<ip>/<share> -U <user>%<password>
smbget -R smb://<ip>/<share> --no-pass           # anonymous
```

---

## Mount a Share

```bash
# Mount to a local directory
sudo mount -t cifs //<ip>/<share> /mnt/smb
sudo mount -t cifs //<ip>/<share> /mnt/smb -o username=<user>,password=<password>
sudo mount -t cifs //<ip>/<share> /mnt/smb -o guest    # anonymous

# Unmount
sudo umount /mnt/smb
```

---

## Brute Force

```bash
# crackmapexec — preferred, handles SMBv2/v3 automatically
crackmapexec smb <ip> -u <user> -p /usr/share/wordlists/rockyou.txt
crackmapexec smb <ip> -u users.txt -p /usr/share/wordlists/rockyou.txt

# hydra — often fails with "does not support SMBv1" on modern targets
# use crackmapexec instead if you hit that error
hydra -l <user> -P /usr/share/wordlists/rockyou.txt smb://<ip>
```

> **Hydra SMBv1 error:** `[ERROR] target smb://... does not support SMBv1`  
> Hydra's SMB module only speaks SMBv1. Modern targets (Windows 10+, recent Samba) disable it.  
> **Fix: use `crackmapexec` instead** — it negotiates SMBv2/v3 automatically.

---

## Quick Reference

| Task | Command |
|------|---------|
| List shares (anon) | `smbclient -L //<ip> -N` |
| Connect to share (anon) | `smbclient //<ip>/<share> -N` |
| Connect with creds | `smbclient //<ip>/<share> -U user%pass` |
| Download one file | `get <file>` |
| Download all (interactive) | `prompt` → `mget *` |
| Download all (recursive) | `prompt; recurse; mget *` |
| Upload a file | `put <file>` |
| Read file in terminal | `get <file> /dev/stdout` |
| Full recon | `enum4linux -a <ip>` |
| Check vulns | `nmap -p 445 --script smb-vuln* <ip>` |
