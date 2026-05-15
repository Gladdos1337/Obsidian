Related: [[(THM) Skynet]] | [[1. nmap]] | [[445 open netbios ssn]] | [[LFI (Local File Inclusion)]] | [[RFI (Remote File Inclusion)]] | [[1.2 Automated Discovery]]

# Port 80 - HTTP (Apache 2.4.18)

## Gobuster Results (root)

```
/admin        (301) → /admin/
/config       (301)
/squirrelmail (301) → /squirrelmail/   ← SquirrelMail webmail
```

## Gobuster Results (/45kra24zxs28v3yd)

```
/administrator (301) → Cuppa CMS login
```

## Key URLs

- `http://10.113.130.138/squirrelmail/` — SquirrelMail login (milesdyson / cyborg007haloterminator)
- `http://10.113.130.138/45kra24zxs28v3yd/administrator/` — Cuppa CMS (vulnerable to LFI/RFI)
