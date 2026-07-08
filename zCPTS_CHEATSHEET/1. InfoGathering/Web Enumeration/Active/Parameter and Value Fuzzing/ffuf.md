
Fuzzing parameters may expose unpublished parameters that are publicly accessible. Such parameters tend to be less tested and less secured, so it is important to test such parameters for the web vulnerabilities we discuss in other modules.

```
#GET Subdirectory / Parameter Fuzzing
ffuf -w params.txt -u "http://TARGET/page.php?FUZZ=test"

#Value Fuzzing (known param, unknown value)
ffuf -w values.txt -u "http://TARGET/page.php?id=FUZZ"

#POST Data Fuzzing
ffuf -w wordlist.txt -X POST -d "username=admin&password=FUZZ" \ -H "Content-Type: application/x-www-form-urlencoded" \ 
-u http://TARGET/login

#Header Fuzzing
ffuf -w wordlist.txt -u http://TARGET/ -H "X-FUZZ: test"

#Multiple FUZZ points (e.g. login brute force) ffuf -w users.txt:FUZZ1 -w passwords.txt:FUZZ2 \ -X POST -d "username=FUZZ1&password=FUZZ2" \ -H "Content-Type: application/x-www-form-urlencoded" \ -u http://TARGET/login -fc 401
```

Default mode is `clusterbomb` (all combos); `-mode pitchfork` pairs wordlists line-by-line instead.