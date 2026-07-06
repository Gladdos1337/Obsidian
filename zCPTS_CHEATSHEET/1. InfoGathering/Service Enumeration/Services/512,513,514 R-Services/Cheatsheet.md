```
#Footprinting
sudo nmap -sV -p 512,513,514 10.0.17.2

#Logging in Using Rlogin
rlogin 10.0.17.2 -l htb-student

#listing users using rwho
rwho

#Listing Authetnicated Users Using Rusers
rusers -al 10.0.17.5
```

The primary concern for `r-services`, and one of the primary reasons `SSH` was introduced to replace it, is the inherent issues regarding access control for these protocols. R-services rely on trusted information sent from the remote client to the host machine they are attempting to authenticate to. By default, these services utilize [Pluggable Authentication Modules (PAM)](https://web.archive.org/web/20241102161436/https://debathena.mit.edu/trac/wiki/PAM) for user authentication onto a remote system; however, they also bypass this authentication through the use of the `/etc/hosts.equiv` and `.rhosts` files on the system. The `hosts.equiv` and `.rhosts` files contain a list of hosts (`IPs` or `Hostnames`) and users that are `trusted` by the local host when a connection attempt is made using `r-commands`. Entries in either file can appear like the following:

**Note:** The `hosts.equiv` file is recognized as the global configuration regarding all users on a system, whereas `.rhosts` provides a per-user configuration.




R-Services are a suite of services hosted to enable remote access or issue commands between Unix hosts over TCP/IP. Initially developed by the Computer Systems Research Group (`CSRG`) at the University of California, Berkeley, `r-services` were the de facto standard for remote access between Unix operating systems until they were replaced by the Secure Shell (`SSH`) protocols and commands due to inherent security flaws built into them. Much like `telnet`, r-services transmit information from client to server(and vice versa.) over the network in an unencrypted format, making it possible for attackers to intercept network traffic (passwords, login information, etc.) by performing man-in-the-middle (`MITM`) attacks.

`R-services` span across the ports `512`, `513`, and `514` and are only accessible through a suite of programs known as `r-commands`. They are most commonly used by commercial operating systems such as Solaris, HP-UX, and AIX. While less common nowadays, we do run into them from time to time during our internal penetration tests so it is worth understanding how to approach them.

The [R-commands](https://en.wikipedia.org/wiki/Berkeley_r-commands) suite consists of the following programs:

- rcp (`remote copy`)
- rexec (`remote execution`)
- rlogin (`remote login`)
- rsh (`remote shell`)
- rstat
- ruptime
- rwho (`remote who`)