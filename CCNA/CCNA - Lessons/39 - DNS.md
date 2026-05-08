4.3 Explain the role of DHCPand DNSwithin the network. 

Don't need to know DNS in depth for CCNA.
Just need to answer DNS related questions.

Things covered:

Purpose of the DNS
Basic functions of DNS
Configuring DNS in Cisco IOS

Purpose - resolving (==converting==) human-readable names to IP adr.

Important - 1.10 Verify IP parameters for Client OS (Windows, Mac OS, Linux)

![[Pasted image 20251215165848.png]]

Shows DNS server.

![[Pasted image 20251215170006.png]]

You don't have to use nslookup, if you ping the server the IP will be shown.

![[Pasted image 20251215170042.png]]

![[Pasted image 20251215170152.png]]

![[Pasted image 20251215170315.png]]

DNS uses both TCP and UDP.
Usually UDP is used, TCP is used for DNS messages > 512 bytes, both use port ==53==

**DNS Cache**
Flushing DNS Cache: wincmd: ==ipconfig /flushdns==
Showing DNS Cache: wincmd: ==ipconfig /displaydns==

**Host file**
Located in Windows\System32\drivers\etc\hosts
You can manually add host locally like this, this isnt DNS, but simple replacement to be used locally.

**Configuring DNS in Cisco IOS**

- For hosts in a network to use DNS, you don't need to configure DNS on the routers, they will simply forward the DNS message like any other packets.
- Cisco router ***can*** be configured as a DNS server, although it's rare., because if internal DNS server is used, usually it's a Win/Linux server.

![[Pasted image 20251215172201.png]]

![[Pasted image 20251215172443.png]]

1. PC1 > R1 Whats the IP adr of youtube.com
2. R1 > 8.8.8.8 Whats the IP adr of youtube.com 
3. dns.google.com *8.8.8.8* replies with IP adr
4. R1 tells PC1 the IP adr
5. ping the youtube.com
6. reply from youtube.com

Configuring Cisco router as DNS server:
![[Pasted image 20251215173021.png]]
Optional command:
![[Pasted image 20251215173135.png]]

All commands needed:

![[Pasted image 20251215173148.png]]

Lab:
![[Day+38+Lab+-+DNS.pkt]]