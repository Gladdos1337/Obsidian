```
#Web directory fuzzing
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://TARGET/FUZZ

#With extensions
ffuf -w wordlist.txt -u http://TARGET/FUZZ -e .php,.txt,.bak,.html

#Recursive (auto-fuzz discovered directories):
ffuf -w wordlist.txt -u http://TARGET/FUZZ -recursion -recursion-depth 2
```