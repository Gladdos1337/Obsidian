CIA triad - Confidentiality, Integrity, Availabilty

Confidentiality - Systems and data should be accessible only for authorized users
Integrity - Systems and data should be unchaanged and trustworthy
Availability - Systems and data should be accessible and usable by authorized entities when required


Vulnerability - Window
Threat - Rock
Exploit - Person with rock


Common threats

technical threats

**DoS** - affects A from CIA (Availability) - Exploits TCP 3 way handshake, spams with SYN messages, but once the target opens the connection and replies with SYN-ACK, the attacker never closes the connection, therefor filling the TCP connection table.
**DDoS** - attacking from multiple points, group of attacks.
**Spoofing Attacks** - Falsifying idendtity (eg fake source IP or MAC) - spoofing is used in a lot of attacks, for example SYN flood attacks involve spoofing, attacker can spoof their source IP so that the SYN-ACK are not sent back to the attacker. Another example of when spoofing is used is with **DHCP exhauston attack (Dhcp starvation)** - attacker sends countless DHCP Discover messages using SPOOFED mac addresses preventing clients from receiving ip adr (DHCP spoof is also a form of DoS attack).
**Reflection/Amplficiation attacks** - attacker sends spoofed requests to 3rd party servers (**reflectors**). this triggers servers to send responses to target , overwhelming it.
when small request triggers large amount of data, its called an **amplification** attack. (This is both an DoS attack and a Spoof)
**Man in the MIDDLE attacks** MITM - attacker secretly intercepts communications between 2 parties, relaying messages between them. Attacker can change the data without them noticing.
Example is ARP poisoning (Or ARP Spoofing) in which an attacker sends spoofed ARP replies to make hosts send their frames to attacker instead of each other.
**Reconnaissance attacks** - collecting info, not an attack per se, common part is OSINT
**Malware** - harmful programs that infect the target computer and then encrypt files, enable access etc. TYPES of MALWARE:
- Virus -  binds itself to a legitimate program or file. When file is executed, virus is activated.
- Worm - Spreads itself, worms often spread by exploiting vuln in software
- Trojan horse - Disguises itself as legitimate software, often spread via email or malicious websites
- Backdoor - Allows unauthorized users to access infected computer, backdoors are often installed by other types of malware
- Ransomware - encrpyts targets data and requires payment for decrypt 
**Password related attacks** - guessing with the help of OSINT or dictionary attack , using list of common passwords, or bruteforcing - when PROGRAM tries list of passwords. difference between dictionary and bruteforce is human vs program.
**Social Engineering** types:
- Spear Phishing - targeted form of phishing , sending to a specific company for example
- Whaling - targeting big boys
- Smishing - SMS phishing
- Vishing - Voice Phishing
- Pretexting - i guess typical social engineering ?
- Tailgating - entering door behind someone so you get in also


Defending from social engineering :
- user awarenes -  sending test phishing emails to see who falls for it
- user training - user trainings are more formal and include graphs etc
- physical access control - network closets protection!!!!11

Passwords and alternatives
Exam topic 5.4 - Describe security password policy elements, such as management, complexity, and password alternatives (multifactor authentication, certificates, and biometrics).

best practices:
- length - use at least 15 chars
- complexity - include big small lettetrs and special symbols #@%^!
- hard to guess - dont use common words

it is often recommended to change passwords regularly - however there is a growing trend against this for legit reasons.
- no reason to change password if its not compromised
- users will reuse passwords or use worse passwords
so its better to just changed when needed

**Password managers** - tool that stores and manage passwords (bitwarden) , but most modern web browsers have this function already . these days using password manager is considered a best practice :
- length and complexity - users can generate and store long, complex and unique password without having to remember it
- auto-fill 
- encrypted storage
they also often support MFA

**Cisco password hashing** - dont store pw as plaintext, hash it, hash function changes password to a fixed-length string that cannot be reverted. Hash functions are one way. 

enable password - stores as plaintext (type 7)
enable secret - md5
**Note ~ Hashing and encryption are often confused. Whereas hashing is irreversible, encryption is reversible.

Always use enable secret, it was md5 (type 5) but now its type 9 (scrpyt) on modern devices.
to use differnt algo use command :
		enable algorythm-type (5, 8(sha256), or 9)
NSA recommends type 8 but 9 is also good

**~ Note The equivalent commands for configuring a user account are username username algorithm-type algorithm secret password to create a user account and hash its password with the specified algorithm, and username username secret type hash to create a user account with an already-hashed password.**

**MFA** - using multiple forms of verification from user . usually 2, so its usually called 2FA
- Knowledge - **SOMETHiNG YOU KNOW*
	- passwords
	- security questions
- Possession -  **SOMETHiNG YOU HAVE*
	 - id badge
	 - smartphone sms
	 - authenticator app
- Inherence - **SOMETHiNG YOU ARE**
	- facial, palm, fingerprint, eye scan

**Digital certs** - key form of authentication , primarly from websites
entities that want cert they send  send CSR (certificate signing request)  to CA (certificate authority), which will generate cert

**Controlling and monitoring users with AAA** - framework for control and monitor of users (eg Network).

Authentication - process of verifying users idendity
Authorization - process of granting user approperate access and perms
Accounting - logging

Enterprises use AAA server to provide AAA services

ISE - cisco aaa server

these servers typically use one of 2 protocol

RADIUS - open standard protocol on udp 1812 and 1813 ports
TACACS+ - cisco protocl TCP port 49





