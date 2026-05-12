80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST



gobuster dir -u 10.113.130.138 -w /usr/share/wordlists/dirb/common.txt


`gobuster dir -u http://10.113.130.138/45kra24zxs28v3yd -w /usr/share/wordlists/dirb/common.txt`

```
.hta                 (Status: 403) [Size: 279]
.htaccess            (Status: 403) [Size: 279]
.htpasswd            (Status: 403) [Size: 279]
admin                (Status: 301) [Size: 316] [--> http://10.113.130.138/admin/]
config               (Status: 301) [Size: 317] [--> http://10.113.130.138/config/]
css                  (Status: 301) [Size: 314] [--> http://10.113.130.138/css/]
index.html           (Status: 200) [Size: 523]
js                   (Status: 301) [Size: 313] [--> http://10.113.130.138/js/]
server-status        (Status: 403) [Size: 279]
squirrelmail         (Status: 301) [Size: 323] [--> http://10.113.130.138/squirrelmail/]
```

http://10.113.130.138/45kra24zxs28v3yd/administrator/