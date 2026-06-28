
PORT STATE SERVICE VERSION

22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)

| ssh-hostkey:

| 3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)

| 256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)

|_ 256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)

80/tcp open http nginx 1.18.0 (Ubuntu)

|_http-server-header: nginx/1.18.0 (Ubuntu)

|_http-title: Did not follow redirect to [http://devvortex.htb/](http://devvortex.htb/)

gobuster vhost command :

gobuster vhost -u [http://devvortex.htb/](http://devvortex.htb/) -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-large-directories.txt -t 100 --append-domain -xs "400"

Found this, looks like old site, probably has flaws i can exploit:

[http://dev.devvortex.htb/](http://dev.devvortex.htb/)

Enumreated that further and found an admin page:

[http://dev.devvortex.htb/administrator/](http://dev.devvortex.htb/administrator/)

Joomla / Ubuntu / Nginx 1.18.0

# If the Joomla site is installed within a folder

# eg www.example.com/joomla/ then the robots.txt file

# MUST be moved to the site root

# eg www.example.com/robots.txt

# AND the joomla folder name MUST be prefixed to all of the

# paths.

# eg the Disallow rule for the /administrator/ folder MUST

# be changed to read

# Disallow: /joomla/administrator/

#

# For more information about the robots.txt standard, see:

# https://www.robotstxt.org/orig.html

User-agent: *

Disallow: /administrator/

Disallow: /api/

Disallow: /bin/

Disallow: /cache/

Disallow: /cli/

Disallow: /components/

Disallow: /includes/

Disallow: /installation/

Disallow: /language/

Disallow: /layouts/

Disallow: /libraries/

Disallow: /logs/

Disallow: /modules/

Disallow: /plugins/

Disallow: /tmp/

</pre></body></html>

Joomla! 4.2

Potential exploit I found :

[https://www.exploit-db.com/exploits/51334](https://www.exploit-db.com/exploits/51334)

This exploit was pretty straight forward, just entered the url, however i had some dependecies i had to use gemini for ruby to run.

lewis

lewis@devvortex.htb

id: 649

Super User

There's also logan paul user, but normal:

logan@devvortex.htb

Users

[649] lewis (lewis) - lewis@devvortex.htb - Super Users

[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info

Site name: Development

Editor: tinymce

Captcha: 0

Access: 1

Debug status: false

Database info

DB type: mysqli

DB host: localhost

DB user: lewis

DB password: P4ntherg0t1n5r3c0n##

DB name: joomla

DB prefix: sd4fg_

DB encryption 0

Users

[649] lewis (lewis) - lewis@devvortex.htb - Super Users

[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info

Site name: Development

Editor: tinymce

Captcha: 0

Access: 1

Debug status: false

Database info

DB type: mysqli

DB host: localhost

DB user: lewis

DB password: P4ntherg0t1n5r3c0n##

DB name: joomla

DB prefix: sd4fg_

DB encryption 0

[http://dev.devvortex.htb/administrator/index.php](http://dev.devvortex.htb/administrator/index.php)

Creds for lewis worked here.

I've tried messing around here but I couldn't find where I can upload the shell.

Used gemini to point me where php files can be uploaded on Joomla, so I went to systems configuration, had some php files there and I changed error.php to shell - so whenever i type wrong link i get shell.

---

Need to get logan password now when i entered the shell.

So the issue was that I changed on the webiste logans password, but that password was the same as the one on the shell, and since i changed it the hashes updated and i couldnt get the hashcat to work... Had to restart the machine and now everything is fine, i got logan access. Next time need to be careful.

DB user: lewis

DB password: P4ntherg0t1n5r3c0n##

logan:tequieromucho

lewis:$2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u

logan:$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12

2026/06/28 14:30:01 CMD: UID=0 PID=43089 | /usr/sbin/CRON -f

2026/06/28 14:30:01 CMD: UID=0 PID=43091 | /bin/bash /root/.cleanup/cleanup.sh

2026/06/28 14:30:01 CMD: UID=0 PID=43090 | /bin/sh -c /root/.cleanup/cleanup.sh

2026/06/28 14:30:01 CMD: UID=0 PID=43092 | /usr/bin/cp -rp /root/.cleanup/cassiopeia /var/www/dev.devvortex.htb/templates/

^^^ that wasnt it..

I used sudo -l

and got this:

[https://github.com/diego-tella/CVE-2023-1326-PoC](https://github.com/diego-tella/CVE-2023-1326-PoC)

Had to google a crash file example and added it there so i just did !/bin/bash


Important things that I've learned from this box:

- When getting access to a administrator page, easiest way to get a shell is via templates that are usually located in System settings , and after that try for plugins.
- Never change passwords of users on the website itself, if there are hashes to be cracked later it will fuck it up.
- 