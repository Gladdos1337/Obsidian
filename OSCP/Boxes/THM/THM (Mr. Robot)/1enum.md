
---
20 FTP-DATA -> Anonymous login > Stuck.
80 HTTP ->
443 HTTPS ->



---


```
PORT    STATE    SERVICE  VERSION
20/tcp  filtered ftp-data
80/tcp  open     http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open     ssl/http Apache httpd
|_http-server-header: Apache
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).
```

