# ffuf Mastery

> Exam-focused ffuf (Fuzz Faster U Fool) reference for CPTS/OSCP. All commands assume `ffuf` is installed and in PATH.

## Basic Syntax

```bash
ffuf -u <URL> -w <wordlist> [flags]
```

---

## 1. Directory/File Fuzzing

### Quick Wins (Small Wordlists)

```bash
# Common directories (fast)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt

# Common files with extensions
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt,.html,.asp,.aspx,.jsp

# API endpoints
ffuf -u http://target.com/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
```

### Medium Wordlists (Thorough)

```bash
# Directory-List-2.3-medium (good balance of speed vs depth)
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt

# Combined with extensions
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -e .php,.txt,.html

# raft wordlists (large, thorough)
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt
```

### Recursive Scanning

```bash
# Basic recursion (-recursion, -recursion-depth)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -recursion -recursion-depth 3

# Recursion with extensions
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.txt -recursion -recursion-depth 2

# Recursion with custom depth and delay
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -recursion -recursion-depth 4 -rate 50
```

### File Extension Fuzzing

```bash
# Discover what extensions the server accepts
ffuf -u http://target.com/indexFUZZ -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt

# PHP-specific
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php,.php3,.php4,.php5,.phtml,.phar
```

---

## 2. Virtual Host Fuzzing

> Essential for discovering subdomains that resolve to the same IP but serve different content.

```bash
# Standard vhost fuzzing (-H for Host header)
ffuf -u http://target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.target.com" -fs 1234

# With custom user-agent to avoid blocking
ffuf -u http://target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.target.com" -H "User-Agent: Mozilla/5.0" -fs 1234

# Using a larger wordlist for vhosts
ffuf -u http://target.com -w /usr/share/seclists/Discovery/DNS/deepmagic.com-prefixes-top50000.txt -H "Host: FUZZ.target.com" -fs 1234
```

### Key Differences: VHOST vs Subdomain Fuzzing

| Technique | Method | Tool |
|-----------|--------|------|
| **VHOST** | Host header fuzzing | ffuf -H "Host: FUZZ.domain.com" |
| **Subdomain** | DNS resolution | gobuster dns, dnsrecon |
| **Subdomain** | HTTP resolution | gobuster vhost |

---

## 3. Parameter Fuzzing

### GET Parameter Fuzzing

```bash
# Discover GET parameters
ffuf -u http://target.com/page.php?FUZZ=test -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# Parameter value fuzzing (known param, test values)
ffuf -u http://target.com/page.php?id=FUZZ -w /usr/share/seclists/Fuzzing/sqli.txt

# Multiple parameters
ffuf -u http://target.com/page.php?p1=FUZZ&p2=FUZZ -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -mode clusterbomb
```

### POST Parameter Fuzzing

```bash
# POST data fuzzing
ffuf -u http://target.com/page.php -X POST -d "username=FUZZ&password=test" -w /usr/share/wordlists/dirb/common.txt -H "Content-Type: application/x-www-form-urlencoded"

# POST JSON fuzzing
ffuf -u http://target.com/api/login -X POST -d '{"username":"FUZZ","password":"test"}' -w /usr/share/seclists/Fuzzing/XSS/XSS-Cheat-Sheet-Payloads.txt -H "Content-Type: application/json"

# Cookie fuzzing
ffuf -u http://target.com/dashboard -b "session=FUZZ" -w /usr/share/seclists/Fuzzing/sqli.txt
```

---

## 4. Filtering Results

> Master filtering to ignore noise. Always baseline your target first.

### Get Baseline (Find Normal Response Size)

```bash
# Single request to establish baseline
curl -s -o /dev/null -w "%{size_download}" http://target.com/

# Or let ffuf do it with -ac (autocalibration)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -ac
```

### Filter Flags Reference

| Flag | Filters By | Example |
|------|-----------|---------|
| `-fs` | Response size (bytes) | `-fs 1234` |
| `-fc` | HTTP status code | `-fc 403,404` |
| `-fw` | Number of words | `-fw 287` |
| `-fl` | Number of lines | `-fl 15` |
| `-mr` | Regex match (include) | `-mr "admin\|dashboard"` |
| `-ms` | Response size match (include) | `-ms 500-1000` |

### Practical Filtering Examples

```bash
# Filter out 404s - most common
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404

# Filter out multiple status codes
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 403,404,301,302

# Filter by response size (removes "not found" pages)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fs 1234

# Filter by word count
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fw 287

# Positive filtering - only show responses matching regex
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -mr "admin"

# Multiple filters combined
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404 -fs 0 -fs 1234
```

---

## 5. Rate Limiting & Performance

> Essential for avoiding WAF blocks and IP bans.

### Rate Control Flags

| Flag | Description | Example |
|------|-------------|---------|
| `-t` | Thread count (default 40) | `-t 10` |
| `-rate` | Requests per second | `-rate 100` |
| `-delay` | Delay between requests | `-delay 100ms` |

### Stealth Configuration

```bash
# Slow and stealthy
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 1 -rate 5

# Moderate speed (good for production)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 10 -rate 30

# Aggressive (use cautiously)
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 50 -rate 200

# With random delay to avoid pattern detection
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 5 -delay 200ms
```

### Random Agent Rotation

```bash
# Use a list of user-agents
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -H "User-Agent: FUZZ" -w /usr/share/seclists/Fuzzing/User-Agents/user-agents.txt -mode pitchfork
```

---

## 6. Advanced Techniques

### Multiple Wordlists (Mode Options)

```bash
# pitchfork: iterate both lists simultaneously (like zip)
ffuf -u http://target.com/FUZZ1/FUZZ2 -w list1.txt:FUZZ1 -w list2.txt:FUZZ2 -mode pitchfork

# clusterbomb: Cartesian product (all combinations)
ffuf -u http://target.com/FUZZ1?id=FUZZ2 -w dirs.txt:FUZZ1 -w params.txt:FUZZ2 -mode clusterbomb

# sniper: single position, try each wordlist
ffuf -u http://target.com/FUZZ -w list1.txt:FUZZ -w list2.txt:FUZZ -mode sniper
```

### Burp Suite Integration

```bash
# Capture request in Burp, save to file, then:
ffuf -request request.txt -request-proto http -w /usr/share/wordlists/dirb/common.txt

# With Burp proxy for debugging
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -replay-proxy http://127.0.0.1:8080

# Full Burp integration example
# 1. Right-click request in Burp -> Copy to file
# 2. Replace what you want to fuzz with FUZZ
ffuf -request burp-request.txt -request-proto https -w wordlist.txt -fc 404
```

### Autocalibration (-ac)

```bash
# Automatically detects and filters false positives
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -ac

# Autocalibration with custom baseline
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -ac -acc 404,403,401
```

### POST with Raw Request from File

```bash
# Save full request (including headers/body) to file
# Replace fuzz point with FUZZ
cat request.txt
# POST /api/login HTTP/1.1
# Host: target.com
# Content-Type: application/json
#
# {"user":"FUZZ","pass":"test"}

ffuf -request request.txt -request-proto https -w wordlist.txt
```

### Output and Reporting

```bash
# Save results to JSON for later analysis
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -o results.json -of json

# Output formats: json, csv, ecsv, md, html
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -o results.md -of md
```

---

## 7. Common Wordlist Paths

```bash
# SecLists (install: sudo apt install seclists)
/usr/share/seclists/Discovery/Web-Content/

# Dirb (comes with Kali)
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirb/big.txt

# Dirbuster
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

# Common API endpoints
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Common parameters
/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt

# SQLi payloads
/usr/share/seclists/Fuzzing/SQLi/Generic-SQLi.txt

# Subdomains for vhost
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## 8. Quick Reference - Most Used Commands

```bash
# 1. Basic directory scan
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 404

# 2. PHP file discovery
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -e .php -fc 404

# 3. VHOST discovery
ffuf -u http://target.com -w subdomains.txt -H "Host: FUZZ.target.com" -fs 1234

# 4. GET param fuzzing
ffuf -u http://target.com/page.php?FUZZ=1 -w params.txt -fc 404

# 5. POST param fuzzing
ffuf -u http://target.com/login.php -X POST -d "user=FUZZ&pass=test" -w usernames.txt

# 6. Stealth mode
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -t 5 -rate 10 -delay 200ms

# 7. Burp integration
ffuf -request captured.req -request-proto https -w wordlist.txt

# 8. Recursive scan
ffuf -u http://target.com/FUZZ -w /usr/share/wordlists/dirb/common.txt -recursion -recursion-depth 3 -fc 404
```

---

## Related Topics

- [[Web Recon Methodology]] - Full recon workflow
- [[../../3.2 SQL Injection/SQL Injection Complete]] - Parameter fuzzing for SQLi
- [[../../3.7 File Upload Attacks/File Upload Attacks Complete]] - Discovering upload endpoints
