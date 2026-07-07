```
#Fuzz the Host header instead of the URL path - finds vhosts that don't resolve via normal DNS

ffuf -w /usr/share -u http://TARGET/ -H "Host: FUZZ.target.com"



```