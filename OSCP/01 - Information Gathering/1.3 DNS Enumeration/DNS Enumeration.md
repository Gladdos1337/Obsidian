# DNS Enumeration — CPTS Exam Reference

> **Why DNS matters:** DNS misconfigurations are among the quickest paths to sensitive data exposure. Zone transfers, subdomain enumeration, and cache snooping can reveal the entire network layout without sending a single probe to a web server.

---

## 1. dig — The Complete Reference

### Record Types

```
# A Record (IPv4 address)
dig example.com A
dig +short example.com A

# AAAA Record (IPv6 address)
dig example.com AAAA

# MX Record (Mail Exchange)
dig example.com MX
dig +short example.com MX

# NS Record (Name Servers)
dig example.com NS
dig +short example.com NS

# TXT Record (SPF, DKIM, DMARC verification records)
dig example.com TXT
dig +short example.com TXT

# CNAME Record (Canonical Name / alias)
dig www.example.com CNAME

# SOA Record (Start of Authority)
dig example.com SOA

# SRV Record (Service location)
dig _sip._tcp.example.com SRV
dig _ldap._tcp.example.com SRV

# ANY Record (All records — often restricted/blocked)
dig example.com ANY

# AXFR (Zone Transfer — THE critical check)
dig @ns1.example.com example.com AXFR
```

### Advanced dig Usage

```
# Specify specific DNS server
dig @8.8.8.8 example.com A
dig @1.1.1.1 example.com MX

# Reverse DNS (PTR)
dig -x 10.10.10.10
dig +short -x 10.10.10.10

# Trace resolution path
dig +trace example.com

# Short response
dig +short example.com

# Brief response
dig +short +nosplit example.com ANY

# Display ONLY answer section
dig +nocomments +noquestion +noauthority +noadditional +nostats example.com

# Query specific port (different DNS server)
dig @10.10.10.10 -p 5353 example.com

# Bulk queries from file
dig -f domains.txt +short A

# DNSSEC validation
dig example.com DNSKEY
dig example.com RRSIG
```

### DNSSEC Records
```
# DNSKEY — public keys
dig example.com DNSKEY

# RRSIG — signatures
dig example.com RRSIG

# DS — delegation signer
dig example.com DS

# NSEC/NSEC3 — authenticated denial of existence
dig example.com NSEC3PARAM
```

---

## 2. nslookup (Windows/Linux)

### Interactive Mode
```
nslookup
> server 10.10.10.1          # Set DNS server
> set type=any               # Query all records
> example.com                # Query domain
> set type=mx                # Mail servers
> example.com
> set type=ns                # Name servers
> example.com
> 10.10.10.10                # Reverse lookup
> exit
```

### Non-Interactive Mode
```
nslookup -type=a example.com
nslookup -type=mx example.com
nslookup -type=ns example.com
nslookup -type=soa example.com
nslookup -type=any example.com
nslookup -type=ptr 10.10.10.10
```

---

## 3. host (Minimal Quick Checks)

```
host example.com                          # Basic A record
host -t mx example.com                    # MX record
host -t ns example.com                    # NS record
host -t txt example.com                   # TXT record
host -t cname www.example.com             # CNAME
host -a example.com                       # ALL records (equivalent to ANY)
host -l example.com ns1.example.com       # Zone transfer
host 10.10.10.10                          # Reverse DNS
host -v -t axfr example.com 10.10.10.1    # Verbose zone transfer
```

---

## 4. DNS Zone Transfer (AXFR)

### What is a Zone Transfer?
- Primary DNS server transfers entire zone to secondary DNS server
- **If misconfigured, ANYONE can request this**
- Reveals: ALL hosts, including internal/dev/staging servers

### How to Check
```
# dig
dig @ns1.example.com example.com AXFR
dig @10.10.10.1 example.com AXFR
dig @ns1.example.com example.com AXFR +short

# host
host -l example.com ns1.example.com

# nslookup
nslookup -type=axfr example.com ns1.example.com

# fierce
fierce --domain example.com --dns-servers 10.10.10.1

# dnsrecon
dnsrecon -d example.com -t axfr
dnsrecon -d example.com -t axfr -n 10.10.10.1
```

### Zone Transfer Scripting
```
#!/bin/bash
for server in $(dig +short NS $1); do
  echo "Trying $server..."
  dig @$server $1 AXFR | grep -v "; Transfer failed"
done
```

---

## 5. Automated DNS Enumeration

### dnsrecon
```
# Basic
dnsrecon -d example.com

# Zone transfer
dnsrecon -d example.com -t axfr

# Brute force subdomains
dnsrecon -d example.com -D /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t brt

# SRV records (Exchange, SIP, LDAP)
dnsrecon -d example.com -t srv

# Reverse lookup range
dnsrecon -r 10.10.10.0/24 -n 10.10.10.1

# Google scraping
dnsrecon -d example.com -t goo

# Standard enumeration (SOA, NS, MX, TXT)
dnsrecon -d example.com -t std

# JSON output
dnsrecon -d example.com -j output.json
```

### dnsenum
```
# Basic enum with default wordlist
dnsenum example.com

# With custom wordlist
dnsenum -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt example.com

# All features on
dnsenum --enum -f wordlist.txt -r /usr/share/dnsenum/dns.txt example.com
```

### fierce
```
# Basic scan
fierce --domain example.com

# With DNS server
fierce -dns example.com -dns-servers 10.10.10.1

# Wordlist brute
fierce --domain example.com --wordlist /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Range reverse lookup
fierce --domain example.com --range 10.10.10.0/24
```

---

## 6. Subdomain Brute Forcing

### gobuster DNS
```
# Standard
gobuster dns -d example.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50

# With custom DNS server
gobuster dns -d example.com -r 10.10.10.1 -w wordlist.txt -t 50

# Wildcard detection
gobuster dns -d example.com -w wordlist.txt --wildcard
```

### massdns (Fastest DNS Brute Forcer)
```
# Install
git clone https://github.com/blechschmidt/massdns.git && cd massdns && make

# Prepare resolvers list (write good DNS servers to resolvers.txt)
echo "1.1.1.1" > resolvers.txt
echo "8.8.8.8" >> resolvers.txt
echo "8.8.4.4" >> resolvers.txt

# Run
./bin/massdns -r resolvers.txt -t A -o S subdomains.txt -w massdns_output.txt

# Extract valid subdomains
grep -oP '.*(?=\.example\.com\. A)' massdns_output.txt | sort -u
```

### DNS Validator
```
# Validate discovered subdomains
dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 20 -o resolvers.txt
```

---

## 7. Subdomain Discovery from Certificates

```
# crt.sh API
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sort -u

# Remove wildcard entries
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | grep -v "\\*" | sort -u

# Certspotter
curl -s "https://api.certspotter.com/v1/issuances?domain=example.com&include_subdomains=true&expand=dns_names" | jq -r '.[].dns_names[]' | sort -u

# Censys cert search (requires API)
# https://search.censys.io/certificates?q=example.com
```

---

## 8. DNS Poisoning & Spoofing Concepts

### DNS Cache Poisoning
- Attacker inserts forged DNS entries into a resolver's cache
- Victims directed to malicious IPs for legitimate domains
- Mitigation: DNSSEC, randomization of source ports and TXID

### DNS Spoofing (MITM on LAN)
```
# Bettercap DNS spoof
sudo bettercap -eval "set dns.spoof.all true; set dns.spoof.domains example.com; dns.spoof on"

# Ettercap DNS spoof
# Edit /etc/ettercap/etter.dns with:
# example.com A 10.10.10.5
# *.example.com A 10.10.10.5
sudo ettercap -T -M ARP /target// /gateway//
```

### dnsspoof (from dnsiff)
```
# Edit hosts file
echo "10.10.10.5 example.com" > dns_spoof_hosts
sudo dnsspoof -i eth0 -f dns_spoof_hosts
```

---

## 9. DNS Cache Snooping

Determine if a DNS resolver has cached specific records:
```
# Check if example.com is cached
nmap -sU -p 53 --script dns-cache-snoop -sV --script-args 'dns-cache-snoop.domains="/usr/share/nmap/scripts/dns-cache-snoop-domains.txt"' $DNS_SERVER

# Manual dig approach
dig @8.8.8.8 example.com +norecurse
# Non-authoritative answer present? It's cached.
```

---

## 10. DNS in Exams — What Matters Most

### Priority Checklist
```
1. Zone Transfer — ALWAYS check first (quick win)
   dig @ns1 $domain AXFR

2. Basic Record Enumeration
   dig NS, MX, TXT, A, AAAA, SOA

3. Subdomain Brute Force
   gobuster dns + seclists wordlist

4. TXT Records (SPF/DKIM/DMARC — info disclosure)
   dig TXT $domain

5. SRV Records (internal services)
   dig _ldap._tcp.$domain SRV
   dig _kerberos._tcp.$domain SRV
   dig _gc._tcp.$domain SRV

6. Reverse DNS on discovered IP ranges
   dig -x IP
```

### Common DNS Misconfigurations
| Issue | What to Check | Impact |
|-------|---------------|--------|
| Zone transfer enabled | `dig @ns1 $domain AXFR` | Full network map |
| Wildcard entries | Random subdomain resolves | Brute force results unreliable |
| Internal IPs in SPF | `dig TXT $domain \| grep spf` | Network layout disclosure |
| Private IPs in A records | `dig A internal.$domain` | Direct internal access |
| Dynamic DNS enabled | `nmap --script dns-update` | Potential DNS takeover |
| DNSSEC not configured | `dig DNSKEY $domain` | Cache poison risk |
| Zone walking enabled | `dig NSEC3PARAM $domain` | Can enumerate all records |

### Wordlists for DNS Brute
```
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt      # Quick
/usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt     # Better
/usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt    # Thorough
/usr/share/amass/wordlists/namelist.txt                                # Amass bundled
```

---

## 11. DNS Response Analysis

### Identifying Filtered Responses
```
# NXDOMAIN — domain doesn't exist (confirmed negative)
# SERVFAIL — server error (try different NS)
# REFUSED — server refused (try different NS or AXFR blocked)
# No answer — may be filtered or record doesn't exist
# Wildcard — any subdomain returns record
```

### Test for Wildcard DNS
```
# Generate a random subdomain
dig asdf1234-$(date +%s).example.com
# If it resolves — wildcard is active. Brute force still works but
# you must filter results against wildcard IP.
```

---

## Related [[wikilinks]]
- [[01 - Information Gathering/1.1 Passive Recon/Passive Reconnaissance]]
- [[01 - Information Gathering/Nmap Mastery]]
- [[01 - Information Gathering/1.2 Active Recon/Active Reconnaissance]]
- [[01 - Information Gathering/1.4 Service Enumeration/Service Enumeration Compilation]]
