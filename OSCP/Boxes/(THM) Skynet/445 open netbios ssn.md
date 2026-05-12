# Port 445 - SMB (Samba 4.3.11)

## Credentials

| User | Password | Access |
|------|----------|--------|
| milesdyson | `cyborg007haloterminator` | milesdyson share, SquirrelMail |
| milesdyson | `)s{A&2Z=F^n_E.B\`` | (alternate / from share) |

## Key Finding

`notes/important.txt` inside the `milesdyson` share revealed the hidden web path:

```
Secret folder: 45kra24zxs28v3yd
```

→ leads to `http://10.113.130.138/45kra24zxs28v3yd/administrator/` (Cuppa CMS)

## Exploit Path

1. `smbclient -L //10.113.130.138 -N` — list shares
2. Connect to `anonymous` — get password list
3. Brute force milesdyson's SMB password using that list
4. Connect to `milesdyson` share — read `notes/important.txt`
5. Use hidden dir to find Cuppa CMS admin panel
