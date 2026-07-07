- Public-facing target, real domain → DNS subdomain fuzzing (`-u FUZZ.domain.com`) works, because it's using actual public DNS infrastructure.
- Internal/lab target, or one you have an IP for but suspect hosts multiple sites → vhost fuzzing (`-H "Host: FUZZ..."`) against the known IP, because DNS won't help you there at all.

## Vhosts vs. Sub-domains

The key difference between VHosts and sub-domains is that a VHost is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be serving two or more different websites.

`VHosts may or may not have public DNS records.`

```
# Fuzz the Host header instead of the URL path — internal
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt 
-u http://TARGET/ -H "Host: FUZZ.target.com"

# Fuzz public facing target
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt 
-u http://TARGET/
```


```
#Wordlists
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-*.txt
```
