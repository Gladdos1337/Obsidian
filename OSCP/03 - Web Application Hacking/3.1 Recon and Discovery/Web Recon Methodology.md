# Web Recon Methodology

> Complete web application reconnaissance workflow for CPTS/OSCP exams. Follow this order for maximum coverage.

---

## 1. Initial Recon Workflow

```
Target URL
    |
    v
[1] Browser Recon          -> DevTools, Manual inspection
    |
    v
[2] Technology ID           -> whatweb, Wappalyzer, builtwith
    |
    v
[3] CMS Fingerprinting      -> wpscan, joomscan, droopescan
    |
    v
[4] Content Discovery       -> [[ffuf Mastery]], gobuster
    |
    v
[5] Burp Suite Analysis     -> Proxy, Repeater, Intruder
    |
    v
[6] API Discovery           -> Endpoint enumeration, docs
    |
    v
[7] Certificate Analysis    -> CRT.sh, SAN enumeration
```

---

## 2. Burp Suite Setup & Workflow

### Initial Configuration

```bash
# Start Burp Suite from CLI
java -jar -Xmx2g /opt/burpsuite_community.jar

# Or via kali menu
burpsuite
```

### Browser Proxy Setup

```
1. Proxy -> Proxy Settings -> Add listener (127.0.0.1:8080)
2. Install Burp CA certificate:
   - Browse to http://burpsuite
   - Download cacert.der
   - Install in browser trusted roots
3. Set browser proxy to 127.0.0.1:8080
4. Ensure "Intercept is on" for active capture
```

### Burp Workflow for Exams

```bash
# Step 1: Configure target scope
# Target -> Scope -> Add URL prefix (include subdomains)

# Step 2: Spider/crawl the application
# Right-click target -> Spider this host
# Or use passive spider via Proxy

# Step 3: Manual browsing with proxy
# Browse all pages, forms, and functionality

# Step 4: Send interesting requests to Repeater
# Right-click -> Send to Repeater (Ctrl+R)

# Step 5: Use Intruder for automated attacks
# Right-click -> Send to Intruder (Ctrl+I)
```

### Burp Extensions (Must-Have)

| Extension | Purpose | Install |
|-----------|---------|---------|
| **Autorize** | IDOR detection | BApp Store |
| **Logger++** | Advanced request logging | BApp Store |
| **Turbo Intruder** | Faster brute-forcing | BApp Store |
| **JSON Web Tokens** | JWT manipulation | BApp Store |
| **Repeater++** | Enhanced Repeater | BApp Store |
| **Content Type Converter** | Bypass filters | BApp Store |

---

## 3. Browser Developer Tools

### Chrome DevTools Quick Reference

```javascript
// Console shortcuts
$('selector')         // document.querySelector
$$('selector')        // document.querySelectorAll
$_                   // Last evaluated expression
$0-$4                // Selected elements in Elements panel

// Network tab filters
status-code:200
status-code:4..
method:POST
mime-type:application/json
larger-than:1k
```

### Key DevTools Workflow

```bash
# 1. Network Tab
# - Check "Preserve log" (essential for redirects)
# - Filter by "XHR"/"Fetch" for API calls
# - Look for hidden endpoints, tokens, API keys
# - Examine response headers (Server, X-Powered-By)

# 2. Sources Tab
# - Search entire codebase (Ctrl+Shift+F)
# - Look for API keys, endpoints, secrets
# - Debug JavaScript execution

# 3. Application Tab
# - Examine cookies (HttpOnly, Secure, SameSite flags)
# - Check localStorage/sessionStorage
# - Review Service Workers

# 4. Console
# - Look for errors revealing technology stack
# - Test XSS payloads
# - Access global variables
```

### Firefox DevTools Specific

```bash
# Accessibility Inspector - check for info leaks
# Storage Inspector - cookies, local storage, indexedDB
# Security Tab - certificate details, mixed content
```

---

## 4. Technology Identification

### whatweb (CLI)

```bash
# Basic scan
whatweb http://target.com

# Aggressive scanning
whatweb -a 3 http://target.com

# Verbose output
whatweb --verbose http://target.com

# Scan subnet
whatweb 10.10.10.0/24

# Custom headers
whatweb -H "Cookie: session=abc123" http://target.com

# Output to file
whatweb --log-json=whatweb.json http://target.com
```

### Wappalyzer (Browser)

```
Browser extension (Firefox/Chrome):
- Shows technologies in toolbar
- JavaScript frameworks, web servers, analytics
- CMS, e-commerce platforms
- CDN, SSL/TLS providers
```

### BuiltWith (Online)

```
https://builtwith.com - Free tier has limitations
Use for: Framework detection, widget analysis
```

### CLI Tools Summary

```bash
# Detect server headers
curl -s -I http://target.com | head -20

# Full response headers
curl -s -v http://target.com 2>&1 | grep -i "<\|>"

# Check for common tech
curl -s http://target.com | grep -i "wp-content\|joomla\|drupal\|laravel\|express"

# Python-based detection
python3 -c "import requests; r=requests.get('http://target.com'); print(r.headers); print(r.text[:500])"
```

---

## 5. CMS Fingerprinting

### WordPress - wpscan

```bash
# Basic enumeration
wpscan --url http://target.com --enumerate

# Enumerate users
wpscan --url http://target.com --enumerate u

# Enumerate plugins (vulnerable versions)
wpscan --url http://target.com --enumerate vp

# Enumerate themes
wpscan --url http://target.com --enumerate vt

# Enumerate everything (aggressive)
wpscan --url http://target.com --enumerate u,vp,vt,tt,cb,dbe

# With API token for vulnerability data
wpscan --url http://target.com --api-token YOUR_API_TOKEN

# Password brute force
wpscan --url http://target.com --passwords /usr/share/wordlists/rockyou.txt --usernames admin

# WPScan API token: https://wpscan.com/register (free)
```

### Joomla - joomscan

```bash
# Install
gem install joomscan

# Basic scan
joomscan -u http://target.com

# Full scan with verbose
joomscan -u http://target.com -v

# Specific components
joomscan -u http://target.com -ec

# Output to file
joomscan -u http://target.com -o joomla_scan.txt
```

### Drupal - droopescan

```bash
# Install
pip3 install droopescan

# Drupal scan
droopescan scan drupal -u http://target.com

# With specific plugins list
droopescan scan drupal -u http://target.com -e plugins

# Verbose output
droopescan scan drupal -u http://target.com --verbose

# Wordpress scan (limited)
droopescan scan wordpress -u http://target.com

# Silverstripe scan
droopescan scan silverstripe -u http://target.com
```

### CMS Detection Quick Checks

```bash
# WordPress indicators
curl -s http://target.com/wp-admin/ | grep -i wordpress
curl -s http://target.com/wp-json/wp/v2/users/ | jq

# Joomla indicators
curl -s http://target.com/administrator/ | grep -i joomla
curl -s http://target.com/components/ | head -5

# Drupal indicators
curl -s http://target.com/CHANGELOG.txt 2>/dev/null
curl -s http://target.com/node/1 | grep -i drupal
```

---

## 6. API Discovery

### Endpoint Enumeration

```bash
# Common API paths to check
curl http://target.com/api/
curl http://target.com/swagger/
curl http://target.com/api/docs
curl http://target.com/api/v1/
curl http://target.com/graphql
curl http://target.com/docs/
curl http://target.com/openapi.json

# API wordlists with ffuf
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -fc 404
ffuf -u http://target.com/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -fc 404
```

### Swagger/OpenAPI Discovery

```bash
# Standard paths
http://target.com/swagger.json
http://target.com/swagger/v1/swagger.json
http://target.com/api/swagger.json
http://target.com/api/docs
http://target.com/api/documentation
http://target.com/v2/api-docs
http://target.com/v3/api-docs
http://target.com/openapi.json

# Parse discovered spec
curl -s http://target.com/swagger.json | jq '.paths | keys'
curl -s http://target.com/openapi.json | jq '.paths | keys'
```

### API Parameter Fuzzing

```bash
# Common API parameters
ffuf -u http://target.com/api/resource?FUZZ=test -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fc 400,404

# API version brute force
ffuf -u http://target.com/api/vFUZZ/ -w /usr/share/seclists/Fuzzing/numbers/1-100.txt -fc 404

# RESTful IDOR testing
for id in $(seq 1 100); do curl -s "http://target.com/api/users/$id"; done
```

### GraphQL Discovery

```bash
# Check for GraphQL
curl -s http://target.com/graphql -X POST -H "Content-Type: application/json" -d '{"query":"query{__typename}"}'

# GraphQL introspection
curl -s http://target.com/graphql -X POST -H "Content-Type: application/json" -d '{"query":"query{__schema{types{name fields{name}}}}"}'

# Introspection query (save to query.graphql)
# Then run:
curl -s http://target.com/graphql -X POST -H "Content-Type: application/json" -d @query.graphql | jq
```

---

## 7. Certificate Analysis

### CRT.sh Certificate Search

```bash
# Search certificates for domain (including wildcards)
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value' | sort -u | grep -v "*"

# Raw CSV format
curl -s "https://crt.sh/?q=%25.target.com&output=csv" | cut -d',' -f5 | sort -u

# Subdomain extraction via crt.sh
curl -s "https://crt.sh/?q=%25.target.com&output=json" | jq -r '.[].name_value[]' | sort -u | tee subdomains.txt

# Recurrence search (search for "target" in any domain)
curl -s "https://crt.sh/?q=target&output=json" | jq -r '.[].name_value' | sort -u
```

### OpenSSL Certificate Inspection

```bash
# Get certificate from live server
openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | openssl x509 -text | head -40

# Extract SAN (Subject Alternative Names)
openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | openssl x509 -text | grep -A 1 "Subject Alternative Name"

# Check certificate dates
openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | openssl x509 -text | grep -E "Not Before|Not After"

# Extract all SAN to file
openssl s_client -connect target.com:443 -servername target.com 2>/dev/null | openssl x509 -text | grep -oP 'DNS:\K[^,]+' | sort -u
```

### Certificate Transparency Logs

```bash
# Using Google's CT search
curl -s "https://certificate.transparency.googleapis.com/v1/query?domain=target.com" | jq

# Facebook CT API
curl -s "https://graph.facebook.com/certificates?query=target.com&fields=domains&limit=10" | jq

# certspotter (passive monitoring)
curl -s "https://api.certspotter.com/v1/issuances?domain=target.com&include_subdomains=true&expand=dns_names" | jq -r '.[].dns_names[]' | sort -u
```

---

## 8. Full Recon Checklist

```
[ ] Port scan all services (nmap -sC -sV -p-)
[ ] Web server banner grab
[ ] Identify CMS/technologies
[ ] Check robots.txt, sitemap.xml
[ ] Directory fuzzing (common + extensions)
[ ] Subdomain/vhost enumeration
[ ] API endpoint discovery
[ ] Parameter fuzzing
[ ] Certificate transparency logs
[ ] Google dorking for the domain
[ ] Wayback Machine for historical endpoints
[ ] Source code review (Ctrl+U in browser)
[ ] Check for exposed .git/.svn directories
[ ] Test default credentials
[ ] Review error pages for info disclosure
```

---

## Related Topics

- [[ffuf Mastery]] - Content discovery tool
- [[../../3.2 SQL Injection/SQL Injection Complete]] - Parameter testing
- [[../../3.3 XSS/XSS Complete]] - Input validation testing
- [[../3.4 LFI RFI/LFI RFI Path Traversal Complete]] - File path testing
