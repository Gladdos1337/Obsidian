**Filtering**

Always request a baseline (e.g. `/thispagedoesnotexist123`) to see what a "soft 404" looks like — status, size, words, lines. Then filter that out.

| Flag  | Meaning                                                          |
| ----- | ---------------------------------------------------------------- |
| `-fc` | filter by **status code** (e.g. `-fc 403,404`)                   |
| `-fs` | filter by **response size**                                      |
| `-fw` | filter by **word count**                                         |
| `-fl` | filter by **line count**                                         |
| `-mc` | **match** status code (default: 200-299,301,302,307,401,403,405) |
| `-ms` | match size                                                       |
| `-mw` | match word count                                                 |

Examples:

```
#kill soft 404 noise:
ffuf -w wordlist.txt -u http://TARGET/FUZZ -fs 4242

#only show 200s
ffuf -w wordlist.txt -u http://TARGET/FUZZ -mc 200
```

**Performance / behavior flags**

| Flag               | Purpose                                                                                  |
| ------------------ | ---------------------------------------------------------------------------------------- |
| `-t`               | threads (default 40) — bump for speed, lower if target is fragile/rate-limits            |
| `-p`               | delay between requests, e.g. `-p 0.1-0.5` (random jitter, helps evade rate-limiting/WAF) |
| `-rate`            | max requests/sec                                                                         |
| `-recursion`       | auto recurse into found directories                                                      |
| `-recursion-depth` | how deep (`-1` = unlimited)                                                              |
| `-maxtime`         | overall time budget                                                                      |
| `-se`              | stop on spurious errors instead of erroring out                                          |
#Report

```
Output/reporting #Report
ffuf -w wordlist.txt -u http://TARGET/FUZZ -o results.json -of json
