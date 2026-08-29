```
#Showing open shares
smbclient -N -L //10.129.14.128

#Connecting with known username/password
smbclient //10.129.42.197/Users -U "fiona%liverpool"

Error: NT_STATUS_IO_TIMEOUT listing \*
~happens when SMBv1 legacy protocol issues, active transfer mode settings, or connection drops hang the connection during share enumeration.

Force SMBv2 / SMBv3:
smbclient //10.129.42.197/Users -U "fiona%liverpool" --max-protocol SMB3

Force SMBv1 (if required by legacy server):
smbclient //10.129.42.197/Users -U "fiona%liverpool" --option="client min protocol=NT1"

#Samba Status
smbstatus

#Footprinting the service
sudo nmap 10.129.14.128 -sV -sC -p139,445

#Logging in into client
rpcclient -U "" 10.129.14.128

#RPClient enum
rpcclient $> srvinfo
rpcclient $> enumdomains
rpcclient $> querydominfo
rpcclient $> netshareenumall
rpcclient $> netsharegetinfo notes

#RPClient user enum
rpcclient $> enumdomusers
>
user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]

queryuser 0x3e9
queryuser 0x3e8

#RPClient group info
querygroup 0x201

#Brute Forcing User RIDs

for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

> An alternative to this would be a Python script from Impacket called samrdump.py.

samrdump.py 10.129.14.128

BE CAREFUL IF ITS DOMAIN OR LOCAL-AUTH USER!!!!!!!!!!!@

#SMBmap
smbmap -H 10.129.14.128

#CrackMapExec
crackmapexec smb 10.129.14.128 --shares -u '' -p ''

#Enum4linux
enum4linux 10.129.14.128 -A

netexec smb 10.129.42.197 -u "user" -p "password" --shares
--local-auth --ignore-pw-decoding
```