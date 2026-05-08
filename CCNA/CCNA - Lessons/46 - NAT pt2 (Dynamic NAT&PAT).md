**4.1 Configure and verify inside source NAT using static and pools**

In dynamic NAT, the router DYNAMICALLY maps *inside local* addresses to *inside global addresses* as needed.

==**An ACL is used to identify which traffic should be translated.**==
- If the SOURCE IP is PERMITTED; the SOURCE IP ==will be== translated
- If the SOURCE IP is DENIED; the SOURCE IP ==will NOT be== translated - **!!!important!!! - the traffic will NOT be dropped, it just means that it won't be translated.**
-  Although they are dynamically assigned, the mappings are still **one-to-one** (one INSIDE LOCAL IP ADDRESS per INSIDE GLOBAL IP ADDRESS)
- If there aren't enough *inside global IP addresses* available (=all are currently being used), it's called "**NAT POOL EXHAUSTION**".
![[Pasted image 20251216193709.png]]
![[Pasted image 20251216193737.png]]
- If a PACKET from another INSIDE HOST arrives and needs NAT but there are no
  AVAILABLE ADDRESSES, the ROUTER will drop the PACKET
- The HOST will be unable to access OUTSIDE NETWORKS until one of the INSIDE
  GLOBAL IP ADDRESSES becomes available
- DYNAMIC NAT entries will time out automatically **if not used**, or you can clear them manually
![[Pasted image 20251216194335.png]]
192.168.0.167 TIMES OUT and 192.168.0.98 is assigned it’s TRANSLATED SOURCE IP

Basically - difference between Static and Dynamic NAT, even though they do the same job, Static ones are permanent, dynamic NAT maps are temp, however hosts still can't use same IP address in the same time, to do that, we need to use **PAT** (port address translation).

But first, lets do dynamic NAT config:
1.
![[Pasted image 20251216194550.png]]
2.![[Pasted image 20251216194600.png]]
3.
![[Pasted image 20251216194614.png]]
4.
![[Pasted image 20251216194621.png]]
5.![[Pasted image 20251216194733.png]]
**R1(config)#show ip nat translations**
![[Pasted image 20251216194834.png]]
- these udp and icmp entries will be cleared after **1 minute**!
- but the original dynamic mapping have age time of **24HRS**, and each time the translation is made, the timer resets.
**R1#show ip nat statistics**
![[Pasted image 20251216195108.png]]

**PAT** (aka **NAT overload**)
- Uses port number to make many internal hosts (abt 65,000)
- The router will keep track of which inside local address is using which inside global address and port.
- PAT selects a random source ports.
- Out of the 3 learned, PAT is the most widely used.

Configuration (very close to others, just adding one more thing):

![[Pasted image 20251216195523.png]]
Translations:

![[Pasted image 20251216195543.png]]The difference here is that there are no one-to-one translations, it's **one-to-many**

Additional useful way to configure PAT (more common):

Same part:

![[Pasted image 20251216195657.png]]

New part:

![[Pasted image 20251216195707.png]]
![[Pasted image 20251216195806.png]]


![[Pasted image 20251216195836.png]]

New commands from this lesson:

![[Pasted image 20251216195846.png]]