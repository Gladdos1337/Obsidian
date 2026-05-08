**1.3.c Concepts of PoE
2.1 a Access ports( data & voic)
4.7 Explain the forwarding per-hop behavior for QoS such as classification, marking, queuing, congestion, policing, shaping**


Traditional phones operate over the:
**PSTN** (public switched telephone network) or **POTS** (plain old telephone service)
IP phones use VoIP (voice over IP) to work over internet
IP phones are connected to switch

![[Pasted image 20251220232845.png]]

IP phones have an **internal** **3-port switch**
1. 1 port is **uplink** to the external switch
2. 1 port is **downlink** to the PC
3. 1 port connects internally phone itself

This allows the PC and the IP phone to share a single switch port.  Traffic from the PC passes thru the IP phone to the switch.
It's recommended to separate 'voice' traffic with 'data' traffic in separate VLANs
use command "**switchport voice VLAN**" on interface.
Traffic from the PC will be untagged, but traffic from the phone will be tagged with a VLAN ID

![[Pasted image 20251220231816.png]]
![[Pasted image 20251220232750.png]]
![[Pasted image 20251220232807.png]]

PoE
• PoE allows Power Sourcing Equipment (**PSE**) to provide POWER to Powered Devices (PD) **over** an ETHERNET cable
• Typically, the PSE is a SWITCH and the PDs are IP PHONES, IP CAMERAS, WIRELESS
ACCESS POINTS, etc.
• The PSE receives AC POWER from the outlet, converts it to DC POWER, and supplies that DC POWER to the PDs (**AC>DC**)

![[Pasted image 20251220233009.png]]

!!! TOO much electrical current can damage the devices
PoE has a process to determine if a CONNECTED DEVICE needs power and how much it needs:
- When a DEVICE is connected to a PoE-Enabled PORT, the PSE (**SWITCH**) sends LOW POWER SIGNALS, monitors the response, and determines how much power the PD needs
- If the DEVICE needs POWER, the PSE supplies the POWER to allow the PD to boot
- he PSE continues to monitor the PD and SUPPLY the required amount of POWER (but not too much!)

**PoE config isn't a CCNA subject.!!!**
Power policing - preventing PD;s from drawing too much power
command : **power inline police** OR **power inline police action err-disable**
- configures power policing with default settings:
	- disable the PORT (the port that draws too much power)
	- send a SYSLOG message if a PD draws too much power
the interface will be put into an "error-disabled" state and can be re-enabled with shutdown no shutdown combo.
![[Pasted image 20251220233508.png]]
power inline police action log does NOT shut down the interface if it draws too much power, it will RESTART it and send SYSLOG message.

![[Pasted image 20251220233545.png]]

![[Pasted image 20251220233559.png]]

Intro to **QoS** - VOICE traffic and DATA traffic used to use enterely separated NETWORKS:
- Voice used PSTN
- DATA used IP NETWORK (Enterprise WAN, Internet, ETC)
QoS wasn't necessary as the different kinds of TRAFFIC didn't compete for BANDWIDTH

![[Pasted image 20251220233731.png]]

Modern networks are typically ""CONVERGED"" networks, basically full of bunch of shit, and all share the same IP network.
This enables COST SAVINGS as well as MORE ADVANCED FEATUREs for VOICE and VIDEO TRAFFIC (ex. Collaboration software like Cisco WebEx, MS Teams, etc)
However, different kinds of traffic now have to compete for bandwidth.
QoS is a set of TOOLS used by NETWORK DEVICES to apply different TREATMENT for different packets.
![[Pasted image 20251220234030.png]]

QoS is used to manage the following characteristics of **NETWORK TRAFFIC**:

**Bandwidth**
 - overrall capacity of the link , measured in bits per second
 - qos tools allow you to reserve certain amount of link's badnwidth for speficifc kind of traffic. eg 20% voice traffic, 30% spefici kinds of data trafic, 50% for other traffic.
 **Delay**
-  one-way delay -the amount of time it takes traffic to go from source to destination -
-  two-way delay -he amount of time it takes traffic to go from source to destination, and back ...

![[Pasted image 20251220234442.png]]
**Jitter**
- The variation in ONE-WAY DELAY between packets sent by the **same application**
- ip phones have 'jitter buffer' to provide a fixed delay to audio packets
**Loss**
- the % of packets sent that do not reach destination
- can be caused by **faulty cables** or when device's packet queues get FULL and the device starts ***discarding packets***.
the following standards are recommended for **acceptable interactive audio quality**:
- one-way delay: 150 ms or less
- jitter: 30 ms or less
- loss: 1% or less
if these standards are not met, there could be noticable reduction in the QUALITY of phone call.


**QoS QUEUING**
- If a NETWORK DEVICE receives messages **FASTER** **than it can FORWARD** them out of the appropriate INTERFACE, the MESSAGES are placed in the **QUEUE**
- By default, the QUEUED MESSAGES will be FORWARDED in FIRST IN FIRST OUT (**FIFO**) manner
	- Message will be SENT in the ORDER they are RECEIVED
- If the QUEUE is FULL, new PACKETS will be DROPPED
- This is called **tail drop**

![[Pasted image 20251220235314.png]]

- **Tail drop** is harmful because it can lead to **TCP GLOBAL SYNCHRONIZATION** 
![[Pasted image 20251220235402.png]]
- When the QUEUE fills UP and TAIL DROP occurs, ALL TCP HOSTS sending traffic will SLOW DOWN the rate at which they SEND TRAFFIC
- They will ALL then INCREASE the RATE at which they send TRAFFIC, which rapidly leads to **MORE CONGESTION**, dropped PACKETS, and the process REPEATS:

![[Pasted image 20251220235458.png]]
**RED (RANDOM EARLY DETECTION)** is used to prevent **TAIL DROP**
When the amount of TRAFFIC in the QUEUE reaches certain THRESHOLD, the DEVICE will start RANDOMLY dropping PACKETS from select TCP FLOWS
- The TCP FLOWS reduce the RATE at which traffic is sent, but increase the RATE OF TRANSMISSION at the same time in WAVES.
**RED** vs **WRED (WEIGHTED RANDOM EARLY DETECTION)**
- **RED** treats all traffic the SAME
- **WRED** - improved version of RED, allows you to control which packets are dropped depending on the TRAFFIC class (will be talked about in next day)

