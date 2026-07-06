```
#NS request to the specific nameserver.
dig ns <domain.tld> @<nameserver>

#ANY request to the specific nameserver.
dig any <domain.tld> @<nameserver>

#AXFR request to the specific nameserver.
dig axfr <domain.tld> @<nameserver>

#Subdomain brute forcing.
dnsenum --dnsserver <nameserver> --enum -p 0 -s 0 -o found_subdomains.txt -f /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-110000.txt <domain.tld>
```

|**DNS Record**|**Description**|
|---|---|
|`A`|Returns an IPv4 address of the requested domain as a result.|
|`AAAA`|Returns an IPv6 address of the requested domain.|
|`MX`|Returns the responsible mail servers as a result.|
|`NS`|Returns the DNS servers (nameservers) of the domain.|
|`TXT`|This record can contain various information. The all-rounder can be used, e.g., to validate the Google Search Console or validate SSL certificates. In addition, SPF and DMARC entries are set to validate mail traffic and protect it from spam.|
|`CNAME`|This record serves as an alias for another domain name. If you want the domain [www.hackthebox.eu](http://www.hackthebox.eu) to point to the same IP as hackthebox.eu, you would create an A record for hackthebox.eu and a CNAME record for [www.hackthebox.eu](http://www.hackthebox.eu).|
|`PTR`|The PTR record works the other way around (reverse lookup). It converts IP addresses into valid domain names.|
|`SOA`|Provides information about the corresponding DNS zone and email address of the administrative contact.|

## DNS Zone Transfer (AXFR) Basics

A zone transfer synchronization mechanism copies the authoritative DNS zone file from a **Primary (Master)** server to **Secondary (Slave)** servers to ensure redundancy and load balancing.

- **Protocol / Port:** **TCP Port 53** (Unlike normal DNS queries which use UDP 53).
- **Abbreviation:** **AXFR** (Asynchronous Full Transfer Zone).
- **Authentication:** Servers typically use a shared secret key (`rndc-key`) to validate communication between authorized masters and slaves.

## Master vs. Slave Architecture

|**Server Role**|**Definition**|**Permissions**|
|---|---|---|
|**Primary (Master)**|The original source of truth for the zone data.|Read & Write (Entries created, deleted, or modified here).|
|**Secondary (Slave)**|Backup servers that fetch zone data from a Master.|Read-Only (Can act as a Master to other lower-tier Slaves).|

## The Synchronization Process

Slaves maintain synchronization using the **SOA (Start of Authority)** record parameters:

1. **Refresh Interval:** At a set interval (typically every hour), the Slave queries the Master for its SOA record.
2. **Serial Number Comparison:** The Slave compares its own SOA serial number with the Master's.
3. **Trigger:** If the Master's serial number is **greater** than the Slave's, the datasets are out of sync, and the Slave pulls a full zone update via an AXFR request.