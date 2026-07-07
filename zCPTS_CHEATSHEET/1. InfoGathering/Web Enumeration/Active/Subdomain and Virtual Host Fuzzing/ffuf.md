```
# Fuzz the Host header instead of the URL path — finds vhosts that don't resolve via normal DNS ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://TARGET/ -H "Host: FUZZ.target.com"
```


```
#Wordlists
/usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-*.txt
```
