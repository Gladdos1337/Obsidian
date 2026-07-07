```
#Web directory fuzzing
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://TARGET/FUZZ

#Extension Fuzzing
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ


#With extensions
ffuf -w wordlist.txt -u http://TARGET/FUZZ -e .php,.txt,.bak,.html,.aspx

!Check if the wordlist you provide has dot (.) after "index" so you might not need to add it manually.

#Recursive (auto-fuzz discovered directories):
ffuf -w wordlist.txt -u http://TARGET/FUZZ -recursion -recursion-depth 2
```

```
#Wordlists
/usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
/usr/share/wordlists/seclists/Discovery/Web-Content/raft-large-directories.txt
```