**4.1 Configure and verify inside source NAT using static and pools**

**==RFC 1918==** specifies the following private IPV4 address ranges:
==**10.0.0.8/8 (10.0.0.0 to 10.255.255.255)> A**==
==**172.16.0.0/12 (172.16.0.0 to 172.31.255.255)> B**==
==**192.168.0.0/16 (192.168.0.0 to 192.168.255.255)> C**==

The /12 and /16 aren't typos, they just specify the address ranges. You are free to divide the addresses however we want. (eg. we can have 192.168.1.0 from Class C)

Everyone uses CIDR anyway, so we don't need to worry about classes, they're the thing of the past.

**==Private IP addresses cannot be used over the internet!==**
That's where the **NAT** comes in.

![[Pasted image 20251216121636.png]]

==Public IP addresses **MUST** be **unique**.==

![[Pasted image 20251216121735.png]]

**NETWORK ADDRESS TRANSLATION (NAT)** is used to MODIFY the SOURCE and/or DESTINATION IP addresses of packets.
Most common use is to allow HOSTS with PRIVATE IP addresses to communicate with other HOSTS over the internet.
**==For CCNA we only need SOURCE NAT and how to configure it on Cisco routers!!!==**

Source NAT example:

![[Pasted image 20251216122550.png]]

**Note: when the server 8.8.8.8 sends the ping back, the SOURCE NAT is still being used, it's just translated back to private IP address.**

**Static source NAT** involves STATICALLY configuring one-to-one mappings of PRIVATE IP addresses to PUBLIC IP addresses.
An ***inside local*** IP address is mapped to an ***inside global*** IP address.

![[Pasted image 20251216123028.png]]
PC1 example:
![[Pasted image 20251216123124.png]]
PC2 example:
![[Pasted image 20251216123147.png]]
![[Pasted image 20251216123041.png]]
We basically need to create inside global ip address for each inside local device that want's access to the internet.
Configuring:

![[Pasted image 20251216123317.png]]

R1(config)#**ip nat inside** - Define the inside interfaces connected to internal network.
R1(config-if)#**ip nat outside** - Define the outside interfaces connected to external network.
R1(config)#**ip nat inside source static (inside local) (inside global)**

Since these are public ip addresses, you need to **OWN** them to **USE** them.
![[Pasted image 20251216123752.png]]
For static **NAT** we don't need to pay attention to port numbers.
But in **PAT** they're important.

![[Pasted image 20251216123944.png]]

R1#clear ip nat translation * - removes dynamic entries.
Static do not expire.

R1# show ip nat statistics > shows inside/outside interfaces and some additional info.

![[Pasted image 20251216124140.png]]

• STATIC NAT involves statically configuring one-to-one mappings of **PRIVATE IP ADDRESSES** to **PUBLIC IP ADDRESSES**
• When traffic from the INTERNAL HOST is sent to the OUTSIDE NETWORK, the ROUTER will translate the SOURCE ADDRESS
**==It basically works from both ways.==**