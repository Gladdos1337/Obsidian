#sqlcmd
## Syntax

Bash

```
sqlcmd -S <target> -U <user> -P <password> [options]
```

## Authentication & Connection

|**Method**|**Command**|
|---|---|
|**SQL Authentication**|`sqlcmd -S <IP> -U <user> -P '<password>'`|
|**Windows Authentication (Kerberos/NTLM)**|`sqlcmd -S <IP> -E`|
|**Domain Credentials**|`sqlcmd -S <IP> -U '<domain>\<user>' -P '<password>'`|
|**Specify Port**|`sqlcmd -S <IP>,<port> -U <user> -P '<password>'`|
|**Specify Named Instance**|`sqlcmd -S <IP>\<instance> -U <user> -P '<password>'`|
|**Specify Database**|`sqlcmd -S <IP> -U <user> -P '<password>' -d <database>`|

## Command Flags

- `-S` : Server / Target host (`HOST`, `IP,PORT`, or `HOST\INSTANCE`).
    
- `-U` : Username.
    
- `-P` : Password.
    
- `-E` : Use trusted connection (Windows Authentication).
    
- `-d` : Select target database.
    
- `-Q` : Execute query and immediately exit.
    
- `-q` : Execute query and remain in interactive prompt.
    
- `-i` : Input SQL script file (`sqlcmd ... -i input.sql`).
    
- `-o` : Output file destination (`sqlcmd ... -o results.txt`).
    
- `-h -1` : Suppress headers/column titles in output.
    
- `-W` : Remove trailing spaces from output fields.
    

## Interactive Session Basics

Once connected interactively (`1>`), commands require `GO` on a new line to execute:

SQL

```
1> SELECT @@version;
2> GO
```

To exit interactive mode:

SQL

```
1> QUIT
```

## Common Enumeration Queries

### Version & Context

SQL

```
SELECT @@version;
SELECT SYSTEM_USER;
SELECT DB_NAME();
GO
```

### Privileges & Roles

SQL

```
-- Check if current user is sysadmin (returns 1 if true, 0 if false)
SELECT IS_SRVROLEMEMBER('sysadmin');
GO
```

### List Databases

SQL

```
SELECT name FROM sys.databases;
GO
```

### List Tables in Current Database

SQL

```
SELECT table_name FROM information_schema.tables;
GO
```

## Command Execution via `xp_cmdshell`

### Check Status

SQL

```
SELECT name, value, value_in_use FROM sys.configurations WHERE name = 'xp_cmdshell';
GO
```

### Enable `xp_cmdshell` (Requires `sysadmin`)

SQL

```
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;
GO
```

### Execute System Commands

SQL

```
EXEC xp_cmdshell 'whoami';
GO
```