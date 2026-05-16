# FTP Cheatsheet

## Connecting

```bash
ftp <ip>
ftp <ip> <port>      # non-standard port
```

## Anonymous Login

```bash
# At the username prompt:
Name: anonymous
Password:            # blank, or use your email
```

## Passive vs Active Mode

| Mode | Direction | When to use |
|------|-----------|-------------|
| Passive (PASV) | Client opens data connection | Default; works through NAT/firewalls |
| Active (PORT) | Server opens data connection to client | Use when passive times out |

```bash
# Inside the ftp client — 'passive' is a TOGGLE, not a setter
passive              # toggles between passive and active

# Check current mode
passive              # output will say "Passive mode on" or "Passive mode off"
```

## Downloading Files

```bash
get <filename>                   # single file
mget *                           # all files (prompts per file)
mget *.txt                       # wildcard — only .txt files
prompt                           # toggle per-file confirmation off/on (off = no prompts)
```

## Changing Local Directory

```bash
lcd /tmp             # change the local directory files are saved into
lpwd                 # show current local directory
```

## Binary vs ASCII Mode

```bash
binary               # switch to binary mode (for non-text files — executables, images, zips)
ascii                # switch back to ASCII mode (default)
```

> Always use `binary` mode when transferring executables or archives, or they will be corrupted.

## wget / curl as FTP Alternatives

These are often more reliable than the interactive client, especially when passive mode is flaky.

```bash
# wget — single file
wget ftp://anonymous:@<ip>/file.txt

# wget — recursive download (entire FTP tree)
wget -r ftp://anonymous:@<ip>/

# wget — force active mode (disable passive)
wget --no-passive-ftp -r ftp://anonymous:@<ip>/

# curl — list directory
curl ftp://anonymous:@<ip>/

# curl — download single file
curl -O ftp://anonymous:@<ip>/file.txt

# curl — force active mode
curl --ftp-port - ftp://anonymous:@<ip>/file.txt
```

> `wget` defaults to **passive** mode. Use `--no-passive-ftp` to switch to active.  
> `curl` defaults to **passive** mode. Use `--ftp-port -` to switch to active.

