
```
#Connecting to a specific share
smbclient //10.129.14.128/notes

#Getting a file
smb: \> get prep-prod.txt 
```


RPClient commands

|   |   |
|---|---|
|`srvinfo`|Server information.|
|`enumdomains`|Enumerate all domains that are deployed in the network.|
|`querydominfo`|Provides domain, server, and user information of deployed domains.|
|`netshareenumall`|Enumerates all available shares.|
|`netsharegetinfo <share>`|Provides information about a specific share.|
|`enumdomusers`|Enumerates all domain users.|
|`queryuser <RID>`|Provides information about a specific user.|