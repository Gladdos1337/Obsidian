- Public-facing target, real domain → DNS subdomain fuzzing (`-u FUZZ.domain.com`) works, because it's using actual public DNS infrastructure.
- Internal/lab target, or one you have an IP for but suspect hosts multiple sites → vhost fuzzing (`-H "Host: FUZZ..."`) against the known IP, because DNS won't help you there at all.

```
# Fuzz the Host header instead of the URL path — internal
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt 
-u http://TARGET/ -H "Host: FUZZ.target.com"

# Fuzz public facing target
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt 
-u http://TARGET/
```



```
#Wordlists
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-*.txt
```
