tldr:
for bigger scale networks its better to automate tasks like new deployments, network wide changes, troubleshooting, network wide policy... etc

couple of tools that are used to automate tasks:

SDN (software-defined networking)
ansible
puppet
py scripts
chef
etc

**SDN**

i must be able to answer 2 logical "planes"

**what does a router do ?**
- routes the network (layer3)
- it uses APR to build APR table, mapping IP addresses to MAC addresses.
- It uses syslog
- uses Dynamic Routing Protocols like OSPF EIGRP RIP etc...
- use ssh
- etc

**what does a switch do**

- it forwards within a LAN by examing information layer2 
- uses STP to ensure there are no layer2 loops in the network
- it builds a MAC address table by examining the source MAC adr of frames
- it also uses syslog
- it allows user to connect via SSH

these various functions can be sorted into planes:

- data plane
- control plane
- management plane
![[Pasted image 20260212065924.png]]

- **data plane** (forwarding plane) - all tasks that **forward** user data/traffic from one interface to another are part of data plane.

	it basically just follows orders made by control plane

- **control plane** - doesn't forward, does the work to enable **data planes** operations:

	OSPF - allows router to build routing table
	STP - it informs the switch about which ports should/shouldn't be used to forward frames
	ARP - are used to build ARP table
	
	**control plane functions influence** (but arent directly involved in) the message forwarding process. 
	
	brains - control plane
	muscles - data plane

- **management plane**

	management plane functions - configuring, managing, monitoring network devices - **doesn't** directly affect the forwarding the messages of data plane
	[[SSH and Telnet]]
	Syslog
	SNMP
	NTP

Management/Control plane are usually managed by CPU
Data plane is usually managed by **ASIC** (Application-Specific Integrated Circuit) - Chips built for specific purposes

Example of using a SW:

When a frame is received, the ASIC is responsible for switching logic.
MAC address table is stored in memory called TCAM (Ternary Content-Addressable Memory). - another common name for MAC address table is CAM table.

A simple summary:
● When a device receives control/
management traffic (destined for
itself), it will be processed in the CPU.
● When a device receives data traffic
which should pass through the
device, it is processed by the ASIC for
maximum speed.


****SDN**** - centralizes the **control plane** into an application called **CONTROLLER**
- example - calculating routes > also how much of the control plane is centralized varies greatly.

The controller can interact programmatically with the network using APIs (Application Programming Interface)

Controller (calculates the routing tables centrally) > R1 - R2 - R3 - R4 .. etc
- communication is done via Southbound Interface (**SBI**) - software interface, not physical.

SBI consists of communication protocol and API
- Some examples  of SBI:
- openFlow
- Cisco OpFlex
- Cisco onePK (Open Network Enviroment Platform KIT)
- NETCONF

Northbound interfaces (**NBI**) allow us to interact with the controller, access data it gathers, program it.

SBI - controller and network devices
NBI - controller and APPs

APIs enable data exchange between programs
REST (Representational State Transfer) API (type of API) is used on the controller as an interface for apps to interact with it

Data is sent in structuroal format such as JSON or XML - it makes it easier for programs to use data.

![[Pasted image 20260212073903.png]]
