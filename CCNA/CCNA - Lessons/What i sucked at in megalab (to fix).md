
**PVST+ priorites**

Configure Rapid PVST+ on all Access and Distribution switches.

> **a.** Ensure that the Root Bridge for each VLAN aligns with the HSRP Active router by configuring the lowest possible STP priority.
> 
> **b.** Configure the HSRP Standby Router for each VLAN with an STP priority one increment above the lowest priority.

**2.** Enable PortFast and BPDU Guard on all ports connected to end hosts (including WLC1). Perform the configurations in interface config mode.

**Etherchannel**

Create a Layer-3 EtherChannel between CSW1 and CSW2 using a Cisco-proprietary protocol. Both switches should actively try to form an EtherChannel. Configure the following IP addresses:


**Static and dynamic routing**

**a.** Make the route via G0/1/0 a floating static route by configuring an AD value 1 greater than the default.


**WiFi configuration**

**FTP**
Use FTP on R1 to download a new IOS version from SRV1:

> **a.** Configure R1’s default FTP credentials: username **cisco**, password **cisco**.
> 
> **b.** Use FTP to copy the file **c2900-universalk9-mz.SPA.155-3.M4a.bin** from SRV1 to R1’s flash drive.
> 
> **c.** Reboot R1 using the new IOS file, and then delete the old one from flash.

