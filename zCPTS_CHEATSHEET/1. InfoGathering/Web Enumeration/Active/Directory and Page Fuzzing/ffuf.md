```
#Web directory fuzzing
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://TARGET/FUZZ

#Extension Fuzzing
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ

> use if else fails -> web-extensions-big.txt
!the wordlist has dot (.) after "index".

#Manual Extension Check
ffuf -w wordlist.txt -u http://TARGET/FUZZ -e .php,.txt,.bak,.html,.aspx

#Page Fuzzing with extension (if we know site ends with eg .php)
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php

#Recursive (auto-fuzz discovered directories):
ffuf -w wordlist.txt -u http://TARGET/FUZZ -recursion -recursion-depth 2
```

```
#Wordlists
/usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
```