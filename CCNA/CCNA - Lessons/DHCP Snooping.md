# DHCP Snooping

## Overview

DHCP Snooping is a Layer 2 security feature that acts as a "firewall" for DHCP messages. It ensures that only legitimate DHCP servers can assign IP addresses and that clients aren't spoofing their identity.

> [!NOTE] It only affects DHCP traffic. All other traffic (web, email, etc.) passes through normally.

## Port Roles: Trusted vs. Untrusted

By default, all ports are untrusted when you enable the feature.

|Port Type|Location|Behavior|
|---|---|---|
|**Trusted**|Facing the DHCP Server or Uplink|Allows all DHCP messages.|
|**Untrusted**|Facing end-users (PCs, APs)|Drops unauthorized or suspicious DHCP messages.|

### The "Bouncer" Logic (Why Packets are Dropped)

On an untrusted port, the switch drops a packet if:

1. It is a Server Message (OFFER, ACK, NAK). Users should not be sending these.
2. It is a RELEASE or DECLINE message where the MAC address inside doesn't match the database entry.
3. The Source MAC of the Ethernet frame doesn't match the Client MAC (chaddr) inside the DHCP packet.

## The Binding Database

The switch builds this table dynamically by "snooping" on legitimate handshakes.

**Stored Information:**

- MAC Address
- IP Address
- Lease Time (in seconds)
- VLAN ID
- Interface (e.g., Gi0/1)
    

> [!WARNING] **Exam Trap** The Default Gateway and DNS Server are NOT stored in this database.

## Option 82 (Relay Agent Information)

Cisco switches add Option 82 to DHCP packets by default when snooping is on.

- It tells the server which switch and port the client is on.
- **Untrusted ports:** If a packet already has Option 82 and comes in an untrusted port, the switch drops it.
- **Trusted ports:** Allowed to receive packets with Option 82.
    

## Configuration

### 1. Enable Feature

```
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10,20
```

### 2. Set Trust Boundary

```
SW1(config)# interface g0/1
SW1(config-if)# ip dhcp snooping trust
```

### 3. Starvation Protection (Rate Limiting)

```
SW1(config)# interface range g0/2 - 24
SW1(config-if)# ip dhcp snooping limit rate 10
```

### 4. Disable Option 82 (If required)

Use this command if the DHCP server does not support the Relay Agent Information option or if it causes issues with address assignment.

```
SW1(config)# no ip dhcp snooping information option
```
## Verification

- `show ip dhcp snooping`: Check global status and trust states.
- `show ip dhcp snooping binding`: View the current mappings.
    

## Related Notes

- [[Dynamic ARP Inspection (DAI)]] - Requires this database to work!
- [[IP Source Guard]] - Requires this database to work!