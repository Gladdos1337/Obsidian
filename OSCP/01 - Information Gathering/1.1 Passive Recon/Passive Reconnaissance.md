# Passive Reconnaissance — CPTS Exam Reference

> **Golden rule:** Passive recon leaves NO fingerprints. Do this FIRST, before touching any target. Spend 20-30 minutes gathering everything you can before launching a single packet.

---

## 1. WHOIS Lookups

### Command-Line WHOIS
```
whois example.com
whois 10.10.10.10                    # IP WHOIS
whois -h whois.arin.net 10.10.10.10  # Query ARIN directly
```

### What to Extract
- **Registrar info** — email, phone, name (password reset vectors?)
- **Name servers** — target often hosts its own DNS = more attack surface
- **Registration dates** — domain age (new = less security)
- **Org address/location** — social engineering context

### Web-Based WHOIS
- `whois.domaintools.com`
- `whois.arin.net`
- `www.icann.org/whois`

---

## 2. DNS Reconnaissance

### dig — The Swiss Army Knife

```
dig example.com A                    # IPv4 records
dig example.com AAAA                 # IPv6 records
dig example.com MX                   # Mail servers
dig example.com NS                   # Name servers
dig example.com TXT                  # SPF, DKIM, DMARC
dig example.com CNAME                # Canonical names
dig example.com ANY                  # ALL records (often restricted)
dig @ns1.example.com example.com AXFR # Zone transfer attempt

# Short output
dig +short example.com A

# Reverse lookup
dig -x 10.10.10.10

# Trace the delegation chain
dig +trace example.com
```

### nslookup (Windows/Linux)
```
nslookup example.com
nslookup -type=MX example.com
nslookup -type=any example.com
nslookup 10.10.10.10                 # Reverse lookup
```

### host (Minimal)
```
host example.com
host -a example.com                  # All records
host -l example.com ns1.example.com  # Zone transfer
host -t MX example.com
host 10.10.10.10                     # Reverse
```

### dnsrecon — Automated DNS Enum

```
# Basic enumeration
dnsrecon -d example.com

# Zone transfer attempt
dnsrecon -d example.com -t axfr

# Brute force subdomains
dnsrecon -d example.com -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t brt

# SRV record scan
dnsrecon -d example.com -t srv

# Reverse lookup range
dnsrecon -r 10.10.10.0/24 -n 10.10.10.1
```

### DNS Key Checks
```
# SPF record — check if internal IPs leaked
dig +short example.com TXT | grep "v=spf1"

# DMARC — is email spoofing allowed?
dig +short _dmarc.example.com TXT

# Zone transfer — holy grail of DNS misconfig
dig @ns1.example.com example.com AXFR
```

---

## 3. Subdomain Enumeration

### Sublist3r
```
sublist3r -d example.com -o subdomains.txt
sublist3r -d example.com -p 80,443 -t 10
```

### Amass
```
# Passive only (no API keys)
amass enum -passive -d example.com -o amass_passive.txt

# Active (with API keys configured)
amass enum -active -d example.com -o amass_all.txt

# Subdomain brute forcing
amass enum -brute -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt

# Visualize
amass viz -enum -d example.com -d3
```

### crt.sh (Certificate Transparency)
```
# Via curl
curl -s "https://crt.sh/?q=example.com&output=json" | jq -r '.[].name_value' | sort -u

# Via web
# https://crt.sh/?q=%25.example.com
```

### DNSDumpster
- Web tool: `https://dnsdumpster.com`
- Returns: DNS records, subdomains, MX hosts in a visual map
- Great for a quick overview

### Chaos (ProjectDiscovery)
```
chaos -d example.com -o chaos_subs.txt
```

### Gobuster DNS Brute
```
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50
```

---

## 4. Google Dorking

### Google Operators Reference Table

| Operator | Example | Description |
|----------|---------|-------------|
| `site:` | `site:example.com` | Restrict to domain |
| `inurl:` | `inurl:admin` | URL contains term |
| `intitle:` | `intitle:"index of"` | Page title contains |
| `intext:` | `intext:"password"` | Body text contains |
| `filetype:` | `filetype:pdf` | Specific file format |
| `ext:` | `ext:sql` | File extension |
| `"..."` | `"confidential report"` | Exact phrase |
| `OR` | `admin OR login` | Logical OR |
| `-` | `-php -asp` | Exclude term |
| `*` | `*.*.example.com` | Wildcard |
| `cache:` | `cache:example.com` | Cached version |
| `link:` | `link:example.com` | Pages linking to (deprecated) |
| `related:` | `related:example.com` | Similar sites |
| `info:` | `info:example.com` | Google's info |
| `allinurl:` | `allinurl:login.php` | Multiple URL terms |
| `allintitle:` | `allintitle:admin panel` | Multiple title terms |

### Critical Dork Strings for Exams

```
# Find login pages
site:example.com inurl:login OR inurl:admin OR intitle:login

# Find exposed files
site:example.com filetype:sql OR filetype:bak OR filetype:old OR filetype:conf

# Directory listing
site:example.com intitle:"index of"

# Exposed config
site:example.com ext:env OR ext:cfg OR ext:config

# Error messages (info disclosure)
site:example.com "Warning" "mysql" OR "SQL" OR "Fatal error"

# Internal URLs exposed
site:example.com inurl:internal OR inurl:intranet OR inurl:dev

# Docs/Sheet sharing
site:example.com "docs.google.com" OR "drive.google.com"

# Username/password in plaintext
site:example.com intext:"username" intext:"password"

# Find subdomains
site:*.example.com

# PHPInfo
site:example.com intitle:"phpinfo()" OR inurl:phpinfo.php

# Webcams
inurl:"viewerframe?mode=motion" OR intitle:"webcam 7"
```

### Google Hacking Database (GHDB)
- `https://www.exploit-db.com/google-hacking-database`

---

## 5. Shodan / Censys

### Shodan
```
# Web interface: https://www.shodan.io

# CLI (install shodan CLI)
shodan init SHODAN_API_KEY
shodan count hostname:example.com
shodan search hostname:example.com
shodan host 10.10.10.10

# Useful filters
hostname:"example.com"           # All services on domain
org:"Example Corp"               # All services on org
port:3389                        # RDP open
port:22 country:US               # SSH in US
"default password"               # Find default creds
ssl.cert.issuer.cn:example.com   # Cert-based search
vuln:CVE-2017-0144               # EternalBlue host

# Save results
shodan search --fields ip_str,port,org,hostnames hostname:example.com > shodan_results.txt
```

### Censys
```
# Web: https://search.censys.io

# Useful filters
services.service_name: HTTP services.port: 443
dns.names: example.com
location.country: "United States" and services.port: 3389
```

---

## 6. Metadata Extraction

### theHarvester
```
# Email + subdomain gathering
theHarvester -d example.com -b google,linkedin,bing,yahoo,baidu

# All sources
theHarvester -d example.com -b all -f results.html

# DNS brute force
theHarvester -d example.com -b dns -l 500
```

### WhatWeb
```
# Website fingerprinting
whatweb example.com
whatweb -a 3 example.com                      # Aggressive
whatweb --log-verbose=whatweb.log example.com
```

### Wappalyzer
- Browser extension for tech stack identification
- Use during manual recon

### BuiltWith
- `https://builtwith.com/example.com`
- Framework, CDN, analytics, hosting provider

---

## 7. Wayback Machine & Historical Recon

### Wayback Machine
```
# Web: https://web.archive.org/web/*/example.com

# CLI with waybackurls
waybackurls example.com | tee wayback_urls.txt

# With gau (GetAllUrls)
gau --subs example.com | tee gau_urls.txt
```

### What Historical Data Reveals
- Old JS files with API keys/endpoints
- Development endpoints (`/test`, `/dev`, `/staging`)
- Removed pages still accessible
- Old versions of robots.txt showing hidden paths
- .git folders, backup files, config files

### Filter Historical URLs
```
# Extract endpoints
cat wayback_urls.txt | grep -E "\.php|\.asp|\.jsp|\.aspx" | sort -u

# Find parameters
cat wayback_urls.txt | grep -E "\?.*=" | sort -u

# Find JS files (hidden API calls)
cat wayback_urls.txt | grep "\.js$" | sort -u
```

### urlscan.io
```
# Web: https://urlscan.io/search/#example.com
# Shows full page analysis, DOM, requests, screenshots
```

---

## 8. Additional Passive Sources

### Email Enumeration
```
# hunter.io — find email addresses
hunterio domain example.com

# Phonebook.cz — email/URL/domain intelligence
# https://phonebook.cz

# Have I Been Pwned (API)
# https://haveibeenpwned.com/API/v3
```

### Social Media Recon
```
# LinkedIn — employees, org structure, tech stack
# Twitter — announcements, support channels, tech preferences
# GitHub — code leaks, internal tools, credentials
# Glassdoor — internal culture, benefits info
```

### Technology Stack Discovery (Alternative Tools)
```
# Wappalyzer (browser extension)
# BuiltWith (https://builtwith.com)
# Netcraft (https://sitereport.netcraft.com)
```

### GitHub Recon
```
# Search code for credentials/API keys
gitdorker -q example.com -tf /tools/GitDorker/Dorks/alldorksv3 -o github_results
```

### ASN Enumeration
```
# Find ASN for target
whois -h whois.arin.net 10.10.10.0

# Enumerate all ASN ranges
whois -h whois.radb.net -- '-i origin AS12345' | grep -Eo "([0-9.]+){4}/[0-9]+" | sort -u
```

---

## 9. Passive Recon Workflow (Exam Checklist)

```
Phase 1: WHOIS + DNS
  [] whois example.com
  [] dig +short NS example.com
  [] dig +short MX example.com
  [] dig +short TXT example.com
  [] dig axfr @ns1.example.com example.com  (zone transfer check)

Phase 2: Subdomain Discovery
  [] crt.sh (curl + jq)
  [] sublist3r -d example.com
  [] amass enum -passive -d example.com
  [] dnsrecon -d example.com -t brt -D wordlist.txt
  [] gobuster dns -d example.com

Phase 3: Tech Stack + Content
  [] whatweb example.com
  [] Wappalyzer (browser)
  [] curl -s -I example.com
  [] theHarvester -d example.com

Phase 4: Archive + History
  [] waybackurls example.com | tee urls.txt
  [] gau --subs example.com
  [] cat urls.txt | grep -E "\.js$|api|dev|test" | more

Phase 5: Search Engines
  [] Google dorks (see table above)
  [] Shodan: hostname:"example.com"
  [] urlscan.io
  [] DNSDumpster
```

---

## Related [[wikilinks]]
- [[01 - Information Gathering/Nmap Mastery]]
- [[01 - Information Gathering/1.2 Active Recon/Active Reconnaissance]]
- [[01 - Information Gathering/1.3 DNS Enumeration/DNS Enumeration]]
- [[01 - Information Gathering/1.4 Service Enumeration/Service Enumeration Compilation]]
