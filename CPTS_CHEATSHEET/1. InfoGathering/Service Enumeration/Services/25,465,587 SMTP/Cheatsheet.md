
```
# Footprinting
sudo nmap 10.129.14.128 -sC -sV -p25

# Footprinting - Open Relay Script
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

#Enumerate users
for user in $(cat users.txt); do echo VRFY $user | nc -nv -w 6 $ip 25 ; done

# Querying OIDs using snmpwalk
snmpwalk -v2c -c <community string> <FQDN/IP>

# Bruteforcing community strings of the SNMP service.
onesixtyone -c community-strings.list <FQDN/IP>

# Bruteforcing SNMP service OIDs.
braa <community string>@<FQDN/IP>:.1.*
```

| Client (`MUA`) | `➞` | Submission Agent (`MSA`) | `➞` | Open Relay (`MTA`) | `➞` | Mail Delivery Agent (`MDA`) | `➞` | Mailbox (`POP3`/`IMAP`) |
| -------------- | --- | ------------------------ | --- | ------------------ | --- | --------------------------- | --- | ----------------------- |

- **MUA (Mail User Agent):** The client application that initiates the email.
- **MSA (Mail Submission Agent):** Checks email validity and origin; also known as a **Relay Server** (target for Open Relay attacks if misconfigured).
- **MTA (Mail Transfer Agent):** Software that checks for spam/size, queries DNS for the recipient's IP, and routes the mail.
- **MDA (Mail Delivery Agent):** Receives packets at the destination and drops the finished email into the recipient's mailbox.


**25** - standard SMTP - default server-to-server communcation. Unencrpyted plaintext by default
**587** ESMTP (Submission) - Used for user-to-server mail submission. Utilizes STARTTLS to upgrade plaintext to SSL/TLS.
**465 SMTPS** - Legacy Implicit SSL/TLS encrypted connection port.

