# Offline Password Attacks

## Table of Contents
- [[#Introduction|Introduction]]
- [[#John the Ripper|John the Ripper]]
- [[#Hashcat|Hashcat]]
- [[#LM Hash Cracking|LM Hash Cracking]]
- [[#NT Hash Cracking|NT Hash Cracking]]
- [[#NTLMv2 Cracking|NTLMv2 Cracking]]
- [[#Kerberos Cracking|Kerberos Cracking]]
- [[#Linux /etc/shadow Cracking|Linux /etc/shadow Cracking]]
- [[#Common Formats Cheatsheet|Common Formats Cheatsheet]]
- [[#Rule-Based Attacks|Rule-Based Attacks]]
- [[#Mask Attacks|Mask Attacks]]
- [[#Markov Mode|Markov Mode]]
- [[#Related Notes|Related Notes]]

---

## Introduction

Offline password attacks involve cracking cryptographic hashes on your own machine. Unlike [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks|online attacks]], you don't interact with the target — you just need the hash.

**Advantages:**
- No account lockouts
- No network traffic/IDS alerts
- Can use full GPU power
- Unlimited attempts

**Prerequisites:**
1. A hash (captured, dumped, or relayed)
2. A cracking tool (John or Hashcat)
3. A wordlist (rockyou, SecLists)
4. Optional: GPU drivers (for Hashcat)

---

## John the Ripper

### Basic Usage

```bash
# Default mode (wordlist)
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Single mode (login name mutations)
john --single hash.txt

# Wordlist with rules (mutation)
john --wordlist=wordlist.txt --rules hash.txt

# Show cracked passwords
john --show hash.txt

# Show hashes in a file
john hash.txt --show

# Restore interrupted session
john --restore

# List available formats
john --list=formats
```

### Unshadow (Linux Password Cracking)

Combine `/etc/passwd` and `/etc/shadow` for John:

```bash
# Step 1: Extract both files
# Need read access to both files

# Step 2: Unshadow
unshadow passwd shadow > unshadowed.txt

# Step 3: Crack with John
john --wordlist=/usr/share/wordlists/rockyou.txt unshadowed.txt

# OR with rules
john --wordlist=wordlist.txt --rules unshadowed.txt
```

### Single Crack Mode

```bash
# Uses login name, GECOS, etc. as password guesses
john --single hash.txt

# Example: user "john" with GECOS "John Smith"
# Tries: john, John, JohnSmith, smithJohn, etc.
```

### Wordlist Mode

```bash
# Basic wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Wordlist with rules
john --wordlist=wordlist.txt --rules=best64 hash.txt

# Wordlist with all rules
john --wordlist=wordlist.txt --rules=all hash.txt

# Multiple wordlists
john --wordlist=wordlist1.txt,wordlist2.txt hash.txt
```

### Incremental Mode (Bruteforce)

```bash
# Very slow, exhaustive search
john --incremental hash.txt

# Incremental with specific charset
john --incremental=LowerNum hash.txt
john --incremental=Alpha hash.txt
```

### Common Hash Formats for John

```bash
# Identify format automatically
john hash.txt

# Force specific format
john --format=raw-md5 hash.txt
john --format=nt hash.txt
john --format=krb5tgs hash.txt
john --format=bcrypt hash.txt
```

### John with Specific Shell

```bash
# Only crack hashes for users with valid shells
john --wordlist=rockyou.txt unshadowed.txt

# Show only root password
john --show unshadowed.txt | grep root
```

---

## Hashcat

### Basic Syntax

```bash
hashcat -m <HASH_TYPE> -a <ATTACK_MODE> hash.txt wordlist.txt
```

**Attack modes (`-a`):**
| Mode | Name | Description |
|------|------|-------------|
| 0 | Straight | Wordlist attack |
| 1 | Combination | Combine two wordlists |
| 3 | Brute-force | Mask attack |
| 6 | Hybrid Wordlist + Mask | Wordlist + append mask |
| 7 | Hybrid Mask + Wordlist | Mask + prepend wordlist |

**Main options:**

```bash
hashcat -m <mode> -a <attack> hash.txt wordlist.txt
hashcat -m <mode> -a <attack> hash.txt wordlist.txt -r rule.txt     # With rules
hashcat -m <mode> -a <attack> hash.txt wordlist.txt --show          # Show cracked
hashcat -m <mode> -a <attack> hash.txt wordlist.txt --username      # Strip usernames
hashcat -m <mode> -a <attack> hash.txt wordlist.txt -O              # Optimized kernel
hashcat -m <mode> -a <attack> hash.txt wordlist.txt --force         # Force (ignore warnings)
hashcat -m <mode> -a <attack> hash.txt wordlist.txt --status        # Show status periodically
hashcat -m <mode> -a <attack> hash.txt wordlist.txt --status-timer=1
```

### Performance Options

```bash
# Best performance (-O = optimized, --workload-profile=3 = highest)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -O --workload-profile=3

# Device management
hashcat -I                    # List devices
hashcat -D 1 -d 1 ...         # Select CPU device
hashcat -D 2 -d 1 ...         # Select GPU device

# Restore session
hashcat --restore --session session_name

# Show only cracked
hashcat -m 1000 hash.txt --show
```

### Cracking Types

#### MD5

```bash
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 0 -a 3 hash.txt ?a?a?a?a?a?a?a?a
```

#### SHA1

```bash
hashcat -m 100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### SHA256

```bash
hashcat -m 1400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### SHA512

```bash
hashcat -m 1700 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### NTLM

```bash
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a?a?a
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r best64.rule
```

#### NTLMv2 (from Responder, CrackMapExec)

```bash
# Hash format: username::domain:challenge:HMAC-MD5:blob
hashcat -m 5600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 -a 0 hash.txt rockyou.txt -r best64.rule
```

#### bcrypt

```bash
# bcrypt is very slow — 1+ second per hash typically
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt --force
```

#### Kerberos TGS (Kerberoasting)

```bash
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### AS-REP (ASREP Roast)

```bash
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### WPA (Wi-Fi)

```bash
hashcat -m 22000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

#### Office Documents

```bash
# Office 2010+
hashcat -m 9600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Office 2013+
hashcat -m 9600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt

# Office 2003-2007
hashcat -m 9700 -a 0 hash.txt rockyou.txt

# Office 1997-2003 (old)
hashcat -m 9800 -a 0 hash.txt rockyou.txt
```

#### mscache / Domain Cached Credentials (DCC)

```bash
# DCC (MS Cache v1)
hashcat -m 1100 -a 0 hash.txt rockyou.txt

# DCC2 (MS Cache v2) — common on modern AD
hashcat -m 2100 -a 0 hash.txt rockyou.txt
```

---

## LM Hash Cracking

LM hashes are case-insensitive and split into two 7-character halves — very weak.

### Extract LM from SAM

```bash
# With impacket
impacket-secretsdump -sam SAM -system SYSTEM LOCAL
```

### Crack LM Hashes

```bash
# LM hash format is 32 hex chars (16 bytes)
# First half (7 chars), second half (7 chars) — cracked separately

# John
john --format=lm hash.txt

# Hashcat (Mode 3000)
hashcat -m 3000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## NT Hash Cracking

NT hashes (NTLM) are case-sensitive MD4 of the password.

### Crack Simply

```bash
# John
john --format=nt hash.txt --wordlist=rockyou.txt

# Hashcat
hashcat -m 1000 -a 0 hash.txt rockyou.txt
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r best64.rule
```

### Pass-the-Hash (no cracking needed)

```bash
# With valid hash, you don't need the password
crackmapexec smb 10.10.14.3 -u administrator -H <NTLM_HASH>
impacket-psexec domain.local/administrator@10.10.14.3 -hashes :<NTLM_HASH>
impacket-wmiexec domain.local/administrator@10.10.14.3 -hashes :<NTLM_HASH>
```

---

## NTLMv2 Cracking

Hashes captured from [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks|Responder]].

### Hash Format

```
username::domain:challenge:HMAC-MD5:blob
```

### Crack

```bash
# John
john --format=ntlmv2 hash.txt --wordlist=rockyou.txt

# Hashcat
hashcat -m 5600 -a 0 hash.txt rockyou.txt
hashcat -m 5600 -a 0 hash.txt rockyou.txt -r best64.rule
```

---

## Kerberos Cracking

### Kerberoasting Hashes (TGS)

```bash
# Crack with John
john --format=krb5tgs hash.txt --wordlist=rockyou.txt

# Crack with Hashcat
hashcat -m 13100 -a 0 hash.txt rockyou.txt
```

### AS-REP Roasting Hashes

```bash
# Crack with John
john --format=krb5asrep hash.txt --wordlist=rockyou.txt

# Crack with Hashcat
hashcat -m 18200 -a 0 hash.txt rockyou.txt
```

---

## Linux /etc/shadow Cracking

### Hash Types in shadow

| Prefix | Hash Type | Salted |
|--------|-----------|--------|
| `$1$` | MD5 | Yes |
| `$5$` | SHA-256 | Yes |
| `$6$` | SHA-512 | Yes |
| `$y$` | Yescrypt | Yes |
| `$2y$` | bcrypt | Yes |

### John with unshadow

```bash
unshadow passwd shadow > combined.txt
john --wordlist=rockyou.txt combined.txt
john --wordlist=rockyou.txt --rules combined.txt
```

### Hashcat with SHA-512 ($6$)

```bash
hashcat -m 1800 -a 0 shadow_hash.txt rockyou.txt
```

---

## Common Formats Cheatsheet

| Algorithm | Hashcat Mode | John Format | Description |
|-----------|-------------|-------------|-------------|
| MD5 | 0 | raw-md5 | Generic MD5 |
| MD4 | 900 | raw-md4 | MD4 (used by NTLM) |
| SHA1 | 100 | raw-sha1 | Generic SHA1 |
| SHA256 | 1400 | raw-sha256 | Generic SHA256 |
| SHA512 | 1700 | raw-sha512 | Generic SHA512 |
| **NTLM** | **1000** | **nt** | **Windows NT hash** |
| NTLMv2 | 5600 | ntlmv2 | NetNTLMv2 (Responder) |
| bcrypt | 3200 | bcrypt | Very slow |
| SHA-512 ($6$) | 1800 | sha512crypt | Linux shadow |
| SHA-256 ($5$) | 7400 | sha256crypt | Linux shadow |
| MD5 ($1$) | 500 | md5crypt | Linux shadow |
| TGS-REP (Kerberoast) | 13100 | krb5tgs | Kerberos TGS |
| AS-REP (ASREP Roast) | 18200 | krb5asrep | AS-REP hash |
| WPA/WPA2 | 22000 | wpapbkdf2 | Wi-Fi handshake |
| Office 2010+ | 9600 | oldoffice | Office documents |
| Office 2013+ | 9600 | oldoffice | Office documents |
| MS Cache v2 | 2100 | mscash2 | Domain cached creds |
| MySQL | 200 | mysql | MySQL hash |
| PostgreSQL | 11000 | postgres | PG hash |
| Apache $apr1$ | 1600 | apr1 | htpasswd |
| Cisco IOS | 500 | md5crypt | Cisco type 5 |
| Cisco PIX | 2400 | — | Cisco PIX hash |
| HMAC-SHA1 (SIP) | 4500 | — | SIP auth |

---

## Rule-Based Attacks

Rules mutate words from a wordlist (prefix, suffix, case change, leet speak).

### Hashcat Rules

```bash
# Use built-in rules
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/InsidePro.rule
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/T0XlC-3.rule
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule

# Generate mutated wordlist (no cracking)
hashcat --stdout -r /usr/share/hashcat/rules/best64.rule base.txt > mutated.txt
```

### John Rules

```bash
# Default rule set
john --wordlist=wordlist.txt --rules hash.txt

# Specific rules
john --wordlist=wordlist.txt --rules=best64 hash.txt
john --wordlist=wordlist.txt --rules=all hash.txt

# Generate mutated wordlist
john --wordlist=wordlist.txt --rules --stdout > mutated.txt
```

### Common Rules Examples

```text
Word: password

Rule: $1 $2 $3          → password123
Rule: c                 → Password
Rule: u                 → PASSWORD
Rule: l                 → password
Rule: $!                → password!
Rule: ^!                → !password
Rule: so0               → passw0rd
Rule: ss$               → pa$$word
Rule: c $1 $2 $3 $!     → Password123!
```

---

## Mask Attacks

Bruteforce with specific character positions (replaces incremental mode).

### Hashcat Mask Attack

```bash
# Attack mode -a 3 with masks
hashcat -m 1000 -a 3 hash.txt ?u?l?l?l?l?l?d?d?d
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a?a?a

# Custom mask
hashcat -m 1000 -a 3 hash.txt ?u?l?l?l?d?d?d?d?s

# Increment length (try from 1 to 8)
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a?a?a --increment
```

### Mask Characters

| Symbol | Characters |
|--------|-----------|
| `?l` | abcdefghijklmnopqrstuvwxyz |
| `?u` | ABCDEFGHIJKLMNOPQRSTUVWXYZ |
| `?d` | 0123456789 |
| `?s` | !"#$%&'()*+,-./:;<=>?@[\]^_`{ |
| `?a` | ?l?u?d?s (all) |
| `?b` | 0x00-0xff (all bytes) |

### Custom Charsets

```bash
# -1 to -4 define custom charsets
hashcat -m 1000 -a 3 hash.txt ?1?l?l?l?l?l?d?d?d -1 ?u?d
# ?1 = uppercase + digits at position 1
```

### Hybrid Attacks

```bash
# Mode 6: Wordlist + Mask (append)
hashcat -m 1000 -a 6 hash.txt rockyou.txt ?d?d?d
# Tries: password123, password1234, etc.

# Mode 7: Mask + Wordlist (prepend)
hashcat -m 1000 -a 7 hash.txt ?d?d?d rockyou.txt
# Tries: 123password, 1234password, etc.
```

---

## Markov Mode

Markov mode (Hashcat) improves speed by prioritizing likely character sequences.

```bash
# Enable with --markov-disable (actually disable), or use defaults
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a?a?a --markov-disable  # Slower but thorough
hashcat -m 1000 -a 3 hash.txt ?a?a?a?a?a?a?a?a  # Default: markov enabled (faster)
```

---

## Related Notes

- [[04 - Password Attacks/Bruteforce Tools Reference]]
- [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks]]
- [[04 - Password Attacks/4.3 Hash Cracking/Hash Cracking Reference]]
- [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]]
