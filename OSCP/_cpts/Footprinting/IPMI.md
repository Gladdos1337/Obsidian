set of standardized specs for hardware based host management systems.
can monitor bunch of stuff...


uses port **623 UDP**
Systems that use the IPMI protocol are called Baseboard Management Controllers (BMCs)
usually running linux

enum:

**NMAP:**

```
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

```
PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)
```

**Metasploit**

```
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 
```

During internal penetration tests, we often find BMCs where the administrators have not changed the default password. Some unique default passwords to keep in our cheatsheets include:


|Product|Username|Password|
|---|---|---|
|Dell iDRAC|root|calvin|
|HP iLO|Administrator|randomized 8-character string consisting of numbers and uppercase letters|
|Supermicro IPMI|ADMIN|ADMIN|
It is also essential to try out known default passwords for ANY services that we discover, as these are often left unchanged and can lead to quick wins. When dealing with BMCs, these default passwords may gain us access to the web console or even command line access via SSH or Telnet.