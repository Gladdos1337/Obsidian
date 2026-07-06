
```
#Footprinting the service
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*

#In addition, we can use `--packet-trace` to track the individual packages and inspect their contents manually. We can see that the `RDP cookies` (`mstshash=nmap`) used by Nmap to interact with the RDP server can be identified by `threat hunters` and various security services such as [Endpoint Detection and Response](https://en.wikipedia.org/wiki/Endpoint_detection_and_response) (`EDR`), and can lock us out as penetration testers on hardened networks.
nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n 

#RDP Security Check.pl
sudo cpan
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl 10.129.201.248



```



For an RDP session to be established, both the network firewall and the firewall on the server must allow connections from the outside. If [Network Address Translation](https://en.wikipedia.org/wiki/Network_address_translation) (`NAT`) is used on the route between client and server, as is often the case with Internet connections, the remote computer needs the public IP address to reach the server. In addition, port forwarding must be set up on the NAT router in the direction of the server.