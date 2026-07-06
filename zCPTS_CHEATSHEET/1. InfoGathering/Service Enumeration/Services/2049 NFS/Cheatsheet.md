```
#Footprinting the service
sudo nmap 10.129.14.128 -p111,2049 -sV -sC

#NSEScript (shows eg. contents and it's stats)
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

# Show available NFS shares
showmount -e <IP>

# Mount the specific NFS share.umount ./target-NFS
mount -t nfs <FQDN/IP>:/<share> ./target-NFS/ -o nolock
```

NFS is based on the [Open Network Computing Remote Procedure Call](https://en.wikipedia.org/wiki/Sun_RPC) (`ONC-RPC`/`SUN-RPC`) protocol exposed on `TCP` and `UDP` ports `111`, which uses [External Data Representation](https://en.wikipedia.org/wiki/External_Data_Representation) (`XDR`) for the system-independent exchange of data. The NFS protocol has `no` mechanism for `authentication` or `authorization`. Instead, authentication is completely shifted to the RPC protocol's options. The authorization is derived from the available file system information. In this process, the server is responsible for translating the client's user information into the file system's format and converting the corresponding authorization details into the required UNIX syntax as accurately as possible.

While NFS protocol version 3.0 (`NFSv3`), which has been in use for many years, authenticates the client computer, this changes with `NFSv4`. Here, as with the Windows SMB protocol, the user must authenticate.

|**Version**|**Features**|
|---|---|
|`NFSv2`|It is older but is supported by many systems and was initially operated entirely over UDP.|
|`NFSv3`|It has more features, including variable file size and better error reporting, but is not fully compatible with NFSv2 clients.|
|`NFSv4`|It includes Kerberos, works through firewalls and on the Internet, no longer requires portmappers, supports ACLs, applies state-based operations, and provides performance improvements and high security. It is also the first version to have a stateful protocol.|