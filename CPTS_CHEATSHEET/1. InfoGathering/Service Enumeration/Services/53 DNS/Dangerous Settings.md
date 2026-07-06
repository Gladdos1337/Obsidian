There are many ways in which a DNS server can be attacked. For example, a list of vulnerabilities targeting the BIND9 server can be found at [CVEdetails](https://www.cvedetails.com/product/144/ISC-Bind.html?vendor_id=64). In addition, SecurityTrails provides a short [list](https://web.archive.org/web/20250329174745/https://securitytrails.com/blog/most-popular-types-dns-attacks) of the most popular attacks on DNS servers.

Some of the settings we can see below lead to these vulnerabilities, among others. Because DNS can get very complicated and it is very easy for errors to creep into this service, forcing an administrator to work around the problem until they find an exact solution. This often leads to elements being released so that parts of the infrastructure function as planned and desired. In such cases, functionality has a higher priority than security, which leads to misconfigurations and vulnerabilities.

| **Option**        | **Description**                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `allow-query`     | Defines which hosts are allowed to send requests to the DNS server.            |
| `allow-recursion` | Defines which hosts are allowed to send recursive requests to the DNS server.  |
| `allow-transfer`  | Defines which hosts are allowed to receive zone transfers from the DNS server. |
| `zone-statistics` | Collects statistical data of zones.                                            |
