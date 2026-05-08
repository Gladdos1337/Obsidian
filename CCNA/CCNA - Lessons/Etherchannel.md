

EtherChannel ConfigurationEtherChannel Configuration
● Member interfaces must have matching configurations.
→ Same duplex (full/half)
→ Same speed
→ Same switchport mode (access/trunk)
→ Same allowed VLANs/native VLAN (for trunk interfaces)


**config**

show spanning-tree (if etherchannel works, it will be replaced with po1)

L2 config:

interface range  g0/1-2
channel-group 1 mode PAGP(auto,desirable)LACP-active,passive) on - static etherlink
interface po1
switchport mode trunk
IF COMMAND GETS REJECTED, then we need to use:
switchport trunk encapsulation dot1q
switchport mode trunk

**FOR PAGP, ONE MUST BE DESIRABLE, OTHER AUTO**!!!!!!!!!!!!!!!!!!!


L3 config:

ON L3 SWITCH - ip routing!!
interface range 
no switchport
channel-group 2 mode on
ip add x.x.x.x subnet




