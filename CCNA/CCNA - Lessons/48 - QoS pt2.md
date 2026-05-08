You need to be able to answer the following questions:

- **What is the point of QoS?**
- **What are the limitation of QoS (Layer 2 QoS vs. Layer 3).**
- **What field is used to mark traffic in Layer 2 and Layer 3?**
- **Can you identify the recommended markings for Layer 2's traffic? What about Layer 3?**
- **Make sure you can convert between QoS Markings to DSCP values (IPP values, EF/AFXX, and CoS).**
- **Understand why we have a trust boundary, and where it should be placed.**
- **What are some of the tools we have to deal with a congested network? RED/WRED?**
- **Be able to explain/differentiate some of the different scheduling methods (WRED, LLQ, CBWFQ, Weighted round-robin).**
- **Know the difference between Policing, and shaping, and how each is used.**

**QoS config isn't CCNA topic.**
The purpose of QoS is to give certain kinds of network traffic priority over others during congestion.

**CLASSIFICATION** organizes network traffice (packets) into traffic classes (categories).

Classifying traffic examples:

- ACL - used to identify entrain traffic.
- NBAR - Network Based Application Recognition - preforms *deep packet inspection*, looking beyond the Layer3/4 information up to layer7 to identify the specific kind of traffic.
- In layer 2 and 3 headers thera re spefici filelds used for this purpose.
  - PCP from 802.1Q tag, used to identify high/low priority traffic. ==**Only used when there is a dot1Q tag!!!**== - which means only used on trunk ports. and access ports with VOIP (not normal access ports)
 ![[Pasted image 20251226220132.png]]
  
  ![[Pasted image 20251226220338.png]]

  - **DSCP from IP header - does the same thing as PCP.**
![[Pasted image 20251227141810.png]]
![[Pasted image 20251227141840.png]]
IP precedence isn't used anymore, because it had unused bits. now DSCP has 6 bits that can be used better.

**MORE ABOUT DSCP:**

RFC 2474 (1998) defines the DSCP field, and other "DiffServ" RFCs elaborate on its use.
• With IPP updated to DSCP, new STANDARD MARKINGS had to be decided on by having generally agreed upon STANDARD MARKINGS for DIFFERENT KINDS of TRAFFIC:
 QoS DESIGN and IMPLEMENTATION is simplified.
 QoS works better between ISPs and ENTERPRISES
 etc.
![[Pasted image 20251221154351.png]]

- **DF** (Default Forwarding):
	- Used for best-effort traffic. (like UDP)
	- The DSCP marking for DF is **0**.
![[Pasted image 20251227142427.png]]

- **EF** (Expedited Forwarding):
![[Pasted image 20251227142523.png]]
	- EF is used for traffic that requires low loss/latency/jitter.
	- The DSCP marking for EF is **46**.

- **AF**(Assured Forwarding) defines 4 traffic classes. All packets in the class have the **SAME** priority. Higher **CLASS NUMBER** means higher priority, higher **DROP PRECEDENCE** means that the packet will more likely be dropped.

![[Pasted image 20251227142906.png]]
![[Pasted image 20251227142922.png]]
![[Pasted image 20251221161931.png]]

- **CS** (Class Selector) defines 8 DSCP values for backward compatibility with IPP.
	- The 3 bits that were added for DSCP are set to 0, and the original IPP bits are used to make 8 values.
	
![[Pasted image 20251227151554.png]]![[Pasted image 20251227151637.png]]



**RFC 4954** was developed with the help of Cisco to bring all of these values together and standardize their use.

- The RFC offers many specific recommendations, but the key ones are:
- **Voice traffic : EF** (platinum)
- **Interactive video: AF4x** (gold)
- **Streaming video: AFx3** (gold)
- **High priority data: AF2x** (silver)
- **Best effort: DF** (bronze)

**Trust Boundaries**
- **Just need to know how their work, no need to config for CCNA.**
If an IP phone is connected to SW port, it is recommended to move the trust boundary to the IP Phones.
This is done via configuration on **SW port!!!** connected to the IP PHONE
**If a user marks their PC's traffic with high priority**, the marking will be changed (not trusted)

![[Pasted image 20251227152012.png]]


**Queuing/Congestion management**

When a NETWORK DEVICE receives TRAFFIC at a FASTER PACE than it can FORWARD out
of the appropriate INTERACE, PACKETS are placed in that INTERFACE’S QUEUE as they wait
to be FORWARDED
• When a **QUEUE** becomes **FULL**, PACKETS that don’t FIT in the QUEUE are dropped (Tail Drop)
• RED and WRED DROP PACKETS early to avoid TAIL DROP
![[Pasted image 20251227153946.png]]
An essential part of QoS is the use of **MULTIPLE QUEUES**
- This is where CLASSIFICATION plays role.
- DEVICE can match TRAFFIC based on various factors (like DSCP MARKINGS in the IP HEADER) and the place it in the appropriate QUEUE.
- However, the DEVICE is only able to forward one FRAME out of an INTERFACE at once SO a SCHEDULER is used to decide which QUEUE TRAFFIC to send from EACH QUEUE, in which order, and router FORWARDS packets one at the time.

classification > queueing > scheduling > transmitted
![[Pasted image 20251227154212.png]]
Common scheduling method is weighed round-robin.
**round-robin** - packets are taking from each queue in order, *cyclically*
**weighted** - more data is taken from high prio queues each time the scheduler reaches that queue.
- **CBWFQ** (Class-Based Weighed Fair Queuing) - popular method of scheduling, using a weighted round-robin scheduler, while guaranteeing each queue a certain percentage of interface's bandwidth during congestion. 
![[Pasted image 20251227154600.png]]
Even with all of this, it's not enough for VOICE and VIDEO.
To solve that, we can configure **LLQ** (Low Latency Queuing) designates one (or more) queues as **STRICT PRIOIRTY QUEUES**.
- IF there is traffic in the queue, the scheduler will **ALWAYS** take the next packet from the queue until it is empty, basically prioritizing this traffic until it's completely transmitted.
![[Pasted image 20251227154819.png]]

LLQ can potentially starve other queues, other queues might never get a turn to send traffic, that's why we need **policing**.

**Shaping and Policing**
- **For CCNA, you need to understand what these 2 do.**

Traffic **shaping** and **policing** are both used to control the rate of traffic.

- **Shaping** *BUFFERS* traffic in a queue if the traffic rate goes OVER the configured rate.
- **Policing** *DROPS* traffic if the traffic rate goes over the configured rate. (*policing also has the option of re-marking the traffic instead of dropping it.*)
	 - 'Burst traffic over the configured rate is allowed for a short period of time.'
	 - This accommodates data applications which typically are "*bursty*" in nature. Instead of constant stream of data, they just send a big LOAD.
	 - The amount of allowed burst traffic is configurable.

In both cases, classification can be used to allow for differentiates for different kinds of traffic.
Common use:
![[Pasted image 20251227153633.png]]