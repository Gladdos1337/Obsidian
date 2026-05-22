

candeger
mypasswordforthatjob

ssh 6498

80/tcp    open  http
6498/tcp  open  unknown
65524/tcp open  unknown
iconvertedmypasswordtobinary

synt{a0jvgf33zfa0ez4y}

flag{63a9f0ea7bb98050796b649e85481845}


bash

```bash
cat /etc/crontab
ls -la /etc/cron*
crontab -l
```

To check if the script is writable:

bash

```bash
ls -la /var/www/.mysecretcronjob.sh
```

If writable, check who owns it and what permissions are set. Writable by your user = exploitable.

Also worth knowing — linpeas checks all of this automatically and highlights writable cron scripts. But you need to understand what it's flagging, which you now do.