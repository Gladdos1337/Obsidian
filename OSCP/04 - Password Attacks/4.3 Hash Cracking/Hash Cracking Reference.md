# Hash Cracking Reference

## Table of Contents
- [[#Hashcat Mode Reference Table|Hashcat Mode Reference Table]]
- [[#Quick Cracking Examples|Quick Cracking Examples]]
- [[#Hash Extraction Methods|Hash Extraction Methods]]
- [[#Hash Formats by Source|Hash Formats by Source]]
- [[#Rule-Based Cracking|Rule-Based Cracking]]
- [[#Looping Multiple Rules|Looping Multiple Rules]]
- [[#Performance Tuning|Performance Tuning]]
- [[#Cracking Strategies|Cracking Strategies]]
- [[#Related Notes|Related Notes]]

---

## Hashcat Mode Reference Table

| Hash Type | Hashcat Mode | Example Hash | Speed | Common Use |
|-----------|-------------|--------------|-------|------------|
| **MD5** | **0** | `8743b52063cd84097a65d1633f5c74f5` | Very fast | Generic web apps |
| **MD4** | **900** | `aee4d3f91ebc0a6a2d2a1e3e1e7e8f9e` | Very fast | NTLM predecessor |
| **SHA1** | **100** | `a9993e364706816aba3e25717850c26c9cd0d89d` | Very fast | Git, legacy apps |
| **SHA256** | **1400** | `2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824` | Fast | Modern web apps, certs |
| **SHA512** | **1700** | `ddaf35a193617abacc417349ae20413112e6fa4e89a97ea20a9eeee64b55d39a2192992a274fc1a836ba3c23a3feebbd454d4423643ce80e2a9ac94fa54ca49f` | Fast | Linux shadow |
| **NTLM** | **1000** | `b4b9b02e6f09a9bd760f388b67351e2b` | Very fast | **Windows most common** |
| **NTLMv2** | **5600** | `admin::DOMAIN:1122334455667788:...` | Fast | Responder captures |
| **bcrypt** | **3200** | `$2a$05$LhayLxezLhK1LhWvKxCyLOj0Gi1l0Xj0sHm3qE4yD` | Very slow | Linux shadow, web apps |
| **SHA512 Crypt** | **1800** | `$6$salt$hash` | Slow | Linux shadow |
| **SHA256 Crypt** | **7400** | `$5$salt$hash` | Slow | Linux shadow |
| **MD5 Crypt** | **500** | `$1$salt$hash` | Medium | Linux shadow (legacy) |
| **Kerberos TGS** | **13100** | `$krb5tgs$23$*user$realm$test/spn*$...` | Medium | Kerberoasting |
| **AS-REP** | **18200** | `$krb5asrep$23$user@realm:...` | Medium | AS-REP Roasting |
| **WPA/WPA2** | **22000** | `WPA*01*4d4fe7aac3a2ce*...` | Slow | Wi-Fi handshake |
| **Office 2010+** | **9600** | `$office$2010$...` | Slow | .docx/.xlsx cracking |
| **Office 2013+** | **9600** | Same as 9600 | Slow | .docx/.xlsx cracking |
| **MS Cache v2** | **2100** | `$DCC2$10240#username#hash` | Fast | Domain cached creds |
| **MS Cache v1** | **1100** | `$DCC1$username$hash` | Fast | Legacy cached creds |
| **MySQL** | **200** | `5d2e19393cc5ef67` | Fast | MySQL 3/4 |
| **MySQL SHA1** | **300** | `*A4B6157319038724E3560894F7F932C8886EBFCF` | Fast | MySQL 5+ |
| **PostgreSQL** | **11000** | `md55d2e19393cc5ef67` | Fast | PostgreSQL |
| **Apache $apr1$** | **1600** | `$apr1$salt$hash` | Medium | htpasswd |
| **Cisco Type 5** | **500** | `$1$salt$hash` | Medium | Cisco IOS |
| **Cisco PIX** | **2400** | `pix$hash` | Fast | Cisco PIX/ASA |
| **HMAC-SHA1 (SIP)** | **4500** | `sip$hash` | Fast | SIP auth |
| **Samsung Android** | **5800** | `$samsung$hash` | Slow | Samsung PIN |
| **Android FDE** | **8800** | `$vold$hash` | Slow | Android encryption |
| **DPAPI** | **—** | N/A | Fast | Windows DPAPI |
| **BitLocker** | **22100** | `$bitlocker$0$...` | Slow | BitLocker recovery |
| **PDF 1.1-1.3 (40-bit)** | **10400** | `$pdf$1*...` | Fast | Old PDFs |
| **PDF 1.4-1.6 (128-bit)** | **10500** | `$pdf$2*...` | Medium | Modern PDFs |
| **PKZIP** | **17200** | `$pkzip2$...` | Slow | ZIP archives |
| **7-Zip** | **11600** | `$7z$...` | Slow | 7z archives |
| **RAR3** | **12500** | `$rar3$...` | Slow | RAR archives |
| **RAR5** | **13000** | `$rar5$...` | Very slow | RAR5 archives |

---

## Quick Cracking Examples

### MD5

```bash
# Hash format: 8743b52063cd84097a65d1633f5c74f5
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### SHA1

```bash
# Hash format: a9993e364706816aba3e25717850c26c9cd0d89d
hashcat -m 100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### SHA256

```bash
# Hash format: 2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824
hashcat -m 1400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### SHA512

```bash
# Hash format: ddaf35a193617abacc417349ae20413112e6fa4e89a97ea20a9eeee64b55d39a2...
hashcat -m 1700 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### NTLM (Windows)

```bash
# Hash format: b4b9b02e6f09a9bd760f388b67351e2b
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 1000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

### NTLMv2 (Responder)

```bash
# Hash format: username::domain:challenge:HMAC-MD5:blob
hashcat -m 5600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 5600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule
```

### bcrypt

```bash
# Hash format: $2a$05$LhayLxezLhK1LhWvKxCyLOj0Gi1l0Xj0sHm3qE4yD
# bcrypt is very slow — use small targeted wordlist first
hashcat -m 3200 -a 0 hash.txt targeted_words.txt
hashcat -m 3200 -a 3 hash.txt ?l?l?l?l?l?l?l?l --increment-min=6
```

### Kerberos TGS (Kerberoasting)

```bash
# Hash format: $krb5tgs$23$*user$realm$test/spn*$...
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
hashcat -m 13100 -a 0 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

### AS-REP (ASREP Roast)

```bash
# Hash format: $krb5asrep$23$user@realm:...
hashcat -m 18200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### WPA/WPA2

```bash
# Hash format: WPA*01*4d4fe7aac3a2ce*...
hashcat -m 22000 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

### Office Documents

```bash
# First extract the hash
python3 office2hash.py /usr/share/john/office2john.py target.docx > hash.txt

# Then crack (Office 2010+)
hashcat -m 9600 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

---

## Hash Extraction Methods

### Windows Hashes (SAM)

```bash
# From a dumped SAM + SYSTEM
impacket-secretsdump -sam SAM -system SYSTEM LOCAL

# Live extraction
impacket-secretsdump domain.local/Administrator:Password@10.10.14.3
```

### Linux Hashes (/etc/shadow)

```bash
# Direct extraction
cat /etc/shadow

# From two files — use unshadow
unshadow passwd shadow > unshadowed.txt
```

### NTLMv2 (Responder)

Responder saves captured hashes to:
```bash
/usr/share/responder/logs/SMB-NTLMv2-SP-<IP>.txt
```

### Kerberos Hashes

```bash
# Kerberoasting
impacket-GetUserSPNs -dc-ip 10.10.14.3 domain.local/user:password -request -outputfile hashes.txt

# AS-REP Roasting
impacket-GetNPUsers -dc-ip 10.10.14.3 -request domain.local/ -format hashcat -outputfile hashes.txt
```

### Office Documents

```bash
# John's office2john
python3 /usr/share/john/office2john.py document.docx > hash.txt

# Or standalone
python3 office2hash.py document.docx > hash.txt
```

### PDF

```bash
python3 /usr/share/john/pdf2john.py document.pdf > hash.txt
```

### ZIP/RAR

```bash
python3 /usr/share/john/zip2john.py archive.zip > hash.txt
python3 /usr/share/john/rar2john.py archive.rar > hash.txt
```

### KeePass (KDBX)

```bash
python3 /usr/share/john/keepass2john.py Database.kdbx > hash.txt
# Crack with mode 13400
hashcat -m 13400 -a 0 hash.txt rockyou.txt
```

### SSH Private Keys

```bash
python3 /usr/share/john/ssh2john.py id_rsa > hash.txt
# Crack with mode 22921
hashcat -m 22921 -a 0 hash.txt rockyou.txt
```

---

## Hash Formats by Source

### From Responder

Format stored in `/usr/share/responder/logs/SMB-NTLMv2-SP-<IP>.txt`
```
Administrator::WIN-DC01:1122334455667788:0123456789abcdef0123456789abcdef:0123456789abcdef0123456789abcdef01234567
```

Use mode **5600**.

### From CrackMapExec (SAM dump)

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bd760f388b67351e2b:::
```

The hash is between the colons for mode **1000**.

### From secretsdump (NTDS.dit)

```
domain.local\Administrator:1111:aad3b435b51404eeaad3b435b51404ee:b4b9b02e6f09a9bd760f388b67351e2b:::
```

Use mode **1000** for the NTLM hash.

### From GetUserSPNs (Kerberoasting)

```
$krb5tgs$23$*user$realm$service/SPN$*GUID$encrypted_ticket_data
```

Use mode **13100**.

---

## Rule-Based Cracking

### Hashcat Built-in Rules

```bash
# List all rules
ls /usr/share/hashcat/rules/

# Best 64 rules (most efficient single rule)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# RockYou rules (comprehensive)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/rockyou-30000.rule

# d3ad0ne rules
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/d3ad0ne.rule

# InsidePro rules
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/InsidePro.rule

# T0XlC rules
hashcat -m 1000 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/T0XlC-3.rule
```

### Looping Multiple Rules

```bash
# Bash loop to run through all rules
for rule in /usr/share/hashcat/rules/*.rule; do
    hashcat -m 1000 -a 0 hash.txt rockyou.txt -r "$rule" --status
done

# Or run best rules sequentially
hashcat -m 1000 -a 0 hash.txt rockyou.txt \
  -r /usr/share/hashcat/rules/best64.rule \
  -r /usr/share/hashcat/rules/rockyou-30000.rule \
  -r /usr/share/hashcat/rules/d3ad0ne.rule
```

### Custom Rule Files

Create `myrules.rule`:

```text
$1 $2 $3            # Append 123
c $1 $2 $3 $!       # Capitalize + 123!
so0 ss$             # password → pa$$word
so@                  # a → @
ss$                  # s → $
c so0 ss$           # Capitalize + leet
$1 $2 $3 $4 $5 $6   # Append 123456
$! $@ $#            # Append specials
```

Run with:
```bash
hashcat -m 1000 hash.txt rockyou.txt -r myrules.rule
```

---

## Performance Tuning

### Optimized Kernels

```bash
# -O flag uses optimized kernels (may fail on complex hashes)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -O

# -w sets workload profile (1=low, 2=normal, 3=high, 4=nightmare)
hashcat -m 1000 -a 0 hash.txt rockyou.txt -O -w 3
```

### Device Selection

```bash
# List devices
hashcat -I

# Use GPU 1
hashcat -m 1000 -a 0 hash.txt rockyou.txt -D 2 -d 1

# Use all GPUs
hashcat -m 1000 -a 0 hash.txt rockyou.txt -D 2
```

### Session Management

```bash
# Name your session
hashcat -m 1000 -a 0 hash.txt rockyou.txt --session crack1

# Restore session
hashcat --restore --session crack1

# Show status
hashcat -m 1000 -a 0 hash.txt rockyou.txt --status
```

### Potfile

```bash
# Show cracked hashes
hashcat -m 1000 hash.txt --show

# Remove hash from potfile
# Edit ~/.hashcat/hashcat.potfile

# Use --potfile-disable to skip potfile
hashcat -m 1000 -a 0 hash.txt rockyou.txt --potfile-disable
```

---

## Cracking Strategies

### Strategy 1: Quick Win
1. Wordlist (rockyou.txt)
2. Wordlist + best64.rule
3. Wordlist + rockyou-30000.rule

```bash
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt --status
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt -r best64.rule --status
```

### Strategy 2: Medium Effort
1. Wordlist + all rules
2. Combination attack (`-a 1`)
3. Hybrid (`-a 6` or `-a 7`)

```bash
hashcat -m <mode> hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/*.rule
```

### Strategy 3: Full Bruteforce
1. Incremental mask (`-a 3`)
2. Start with short lengths, expand
3. Use Markov to prioritize

```bash
hashcat -m <mode> -a 3 hash.txt ?a?a?a?a?a?a?a?a --increment --increment-min=6
```

### Priority Order (by speed)

1. **NTLM** (mode 1000) — fastest, billions of attempts/sec
2. **MD5** (mode 0) — very fast
3. **SHA1** (mode 100) — very fast
4. **NTLMv2** (mode 5600) — fast
5. **SHA256** (mode 1400) — fast
6. **SHA512** (mode 1700) — fast
7. **Kerberos** (modes 13100, 18200) — moderate
8. **Linux shadow** (modes 1800, 7400) — moderate-slow
9. **Office** (mode 9600) — slow
10. **bcrypt** (mode 3200) — very slow (1+ hash per second)

---

## Related Notes

- [[04 - Password Attacks/Bruteforce Tools Reference]]
- [[04 - Password Attacks/4.1 Online Attacks/Online Password Attacks]]
- [[04 - Password Attacks/4.2 Offline Attacks/Offline Password Attacks]]
- [[04 - Password Attacks/4.4 Kerberos Attacks/Kerberos Attacks]]
