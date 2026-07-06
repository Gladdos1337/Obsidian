```
# Querying OIDs using snmpwalk
snmpwalk -v2c -c <community string> <FQDN/IP>

# Bruteforcing community strings of the SNMP service.
onesixtyone -c community-strings.list <FQDN/IP>

# Bruteforcing SNMP service OIDs.
braa <community string>@<FQDN/IP>:.1.*
```




`Organizations frequently stick to **SNMPv2c** due to the migration complexity of v3. As a penetration tester, your main objective is to brute-force or sniff these community strings to perform a dictionary walk of the MIB, which leaks dense network configuration data, active processes, system banners, and sometimes internal credentials.`

SNMP is a standard protocol used to monitor, configure, and manage network hardware (routers, switches, servers, IoT) remotely.

- **SNMP Agent Port:** **UDP 161** (Used by the client to query values or send control commands).
- **SNMP Trap Port:** **UDP 162** (Asynchronous notifications sent automatically from the server to the client when a specific event occurs).

## Data Structure: MIB & OID

### MIB (Management Information Base)

- An independent ASCII text file format (written in **ASN.1**) that lists all queryable objects on a device.
- **Purpose:** It does not contain live data; it acts as a map blueprint detailing where information is located, its data type, and access rights.

### OID (Object Identifier)

- A unique, hierarchical address representing a specific node in a tree namespace.
- Written as a dot-separated sequence of integers (e.g., `.1.3.6.1.2.1.1.1.0`). The longer the chain, the more specific the component being queried.

## Protocol Versions & Security Controls

|**Version**|**Authentication Mechanism**|**Encryption**|**Notes / Security Status**|
|---|---|---|---|
|**SNMPv1**|None|None (Plaintext)|Highly insecure. Anyone on the network can read/write data.|
|**SNMPv2c**|**Community Strings** (Cleartext passwords)|None (Plaintext)|Community strings act as weak passwords (`public` / `private`). Easily sniffed over the wire.|
|**SNMPv3**|Traditional Username & Password|**Full Encryption** (via Pre-Shared Keys)|Secure but significantly more complex to configure.|