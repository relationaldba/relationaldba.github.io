Author: Varun Deshpande
Started: 2021-08-16
Last Modified: 2021-08-18
# PostgreSQL Administration





9-07

---

<div id="top"/>

# Table of contents
1. [PostgreSQL Administration](#PostgreSQL-Administration)
	* [Installation on Debian](#Installation-on-Debian)
	* [Initialization of the PostgreSQL cluster](#Initialization-of-the-PostgreSQL-cluster)
	* [Configuring PostgreSQL](#Configuring-PostgreSQL)
	* [Securing PostgreSQL](#Securing-PostgreSQL)
2. [PostgreSQL Internals](#PostgreSQL-Internals)
	* [System Catalogs](#System-Catalogs)
	* [System Functions](#System-Functions)
3. [The SQL Language](#The-SQL-Language)
4. [PostgreSQL Programming](#PostgreSQL-Programming)

---
<br/>
<br/>
<br/>
<br/>
<br/>
<br/>

[top](#top)
<div id="PostgreSQL-Administration"/>

# PostgreSQL Administration
This section will cover the topics that are useful for a DBA in carrying out the day to day administrative tasks like installation, backups, restore, indexing and vacuuming, performance tuning, user/role management and access control.

[top](#top)
<div id="Installation-on-Debian"/>

## Installation on Debian
Follow the below steps to install pre-packaged version of PostgreSQL server on a vanilla Debian server.
### Steps
- From the terminal or command window on your machine, SSH into the Debian server as a user with `sudo` privileges.
	```bash
	$ ssh user_name@server_ip_address
	```
- Update the apt cache and upgrade the packages to the latest version on the server
	```bash
	$ sudo apt update && sudo apt upgrade
	```
- Install PostgreSQL server and client tools on the Debian server. 
	```bash
	$ sudo apt install postgresql-13 postgresql-client
	```
- Optionally, install phpPgAdmin for remote administration of the PostgreSQL server, although these tasks can be performed efficiently using the pgAdmin4 client from your local machine.
	```bash
	$ sudo apt install phppgadmin
	```
- Confirm that the PostgreSQL service is up and running after the installation completes.
	```bash
	$ sudo systemctl status postgresql
	● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2021-08-31 17:16:53 EDT; 16min ago
	```

### User account for PostgreSQL service
The PostgreSQL daemon runs under the Linux operating system user account `postgres` that is created automatically during installation. (Do not confuse the operating system user account `postgres` with the database superuser account `postgres` in the PostgreSQL server). The installer scripts also grant this user complete access to the data directory where the PostgreSQL database files reside (see the defaults below). The `postgres` user should never own the PostgreSQL executable files, to ensure that a compromised server process could not modify those executables. You are not required to do anything if you installed the pre-packaged version of PostgreSQL on Debian.

### Default directories in Debian
|||
|-|-|
| **Data** `$PGDATA`		| `/var/lib/postgresql/13/main/` |
| **Binaries**					|`/usr/lib/postgresql/13/bin/` |
| **Shared Resources** 	| `/usr/share/postgresql/13/` |
| **Config**						| `/etc/postgresql/13/main/` |
| **Logs**							| `/var/log/postgresql/` | 

[top](#top)
<div id="Initialization-of-the-PostgreSQL-cluster"/>

## Initialization of the PostgreSQL cluster
### Manual Initialization
The pre-packaged PostgreSQL server is automatically initialized by the installer during the installation process. It uses the default values to initialize the cluster. **You are not required to manually initialize the PostgreSQL cluster on Debian**, however, you can use the below command to manually initialize PostgreSQL server.
```bash
$ initdb -D /var/lib/postgresql/13/main --auth-local peer --auth-host md5
```
Alternatively, you can run `initdb` via the [pg_ctl](https://www.postgresql.org/docs/13/app-pg-ctl.html "pg_ctl") program:

```bash
$ pg_ctl -D /usr/local/pgsql/data initdb
```

### Start, Stop and Restart PostgreSQL Service
Use the below commands on Debian

**Start PostgreSQL service**
```bash
$ sudo systemctl start postgresql
```

**Stop PostgreSQL service**
```bash
$ sudo systemctl stop postgresql
```

**Re-start PostgreSQL service**
```bash
$ sudo systemctl restart postgresql
```

### Changing configuration
PostgreSQL stores all the config values in the configuration file located at: `/etc/postgresql/13/main/postgresql.conf`. This file consists of lines of the form: `name = value` (the "=" is optional). Whitespace may be used instead of the `=` symbol. Comments are introduced with a `#` symbol anywhere on a line. If a parameter is commented-out, it takes the default value. This file is read on server startup and when the server receives a SIGHUP signal.  If you edit the file on a running system, you have to SIGHUP the server for the changes to take effect, run `pg_ctl reload` on the server terminal, or `SELECT pg_reload_conf()` using the psql client. Some settings require the PostgreSQL service to be restarted to take effect.

In the newer versions of PostgreSQL it is discouraged to edit the default config file `postgresql.conf`, and instead settings should be changed using the `ALTER SYSTEM` commands. PostgreSQL saves the modified settings in a separate file `postgresql.auto.conf` which supersedes the default config file. You should not edit the file as it will be overwritten by the `ALTER SYSTEM` commands. The `ALTER SYSTEM` command takes the below form:
```sql
ALTER SYSTEM SET configuration_parameter { TO | = } { value | 'value' | DEFAULT }
```
You can reset a parameter to its default value by executing
```sql
ALTER SYSTEM RESET configuration_parameter
```
Or reset all the system parameters to their default values by executing 
```sql
ALTER SYSTEM RESET ALL
```
Depending on the type of setting, you will need to reload the config file or restart the service. Refer PostgreSQL documentation for details.

### Recommended configuration
Consider changing the default settings to the widely popular best practices. You will need to restart the PostgreSQL service after changing the configuration values.

| Parameter | Value | Description |
|-|-|-|
| listen_addresses | 'x.x.x.x' | Specify the IP addr that the PostgreSQL service should listen on. Specifying `*` here will make PostgreSQL listen on all the IPs of the system. | 
| port | 5432 | Specify a uncommon port for security purposes. 5432 is the default. |
| password_encryption | scram-sha-256 | Most secure standard of encryption. |
| ssl | on | Allow SSL-encrypted connections to the server. |
| shared_buffers | 75% of system memory | Allow usage of memory for better performance. |
| maintenance_work_mem | 1-2 GB | Faster maintenance activities. |
| log_min_duration_statement | 250ms | Capture statements that run >250ms in the log. |
| log_duration | on | Log the duration of statement execution. |
| log_hostname | on | Log the hostname of the client. |
| timezone | 'Canada/Eastern' | Set the appropriate timezone. |
| autovacuum | on | Enable the autovacuum process. |


### Connecting to PostgreSQL server
The default superuser in the PostgreSQL cluster is the `postgres` database user. This is the only user account that gets created in the PostgreSQL cluster during the installation process. The pre-packaged installer does not set the password for the `postgres` database user, hence you cannot use a client tool like `psql` and connect using a password authentication. However you can login as the superuser by impersonating the account. 
```bash
$ sudo -u postgres psql
psql (13.3 (Debian 13.3-1))
Type "help" for help.

postgres=#
```
By default, the `postgres` user is authenticated using `peer` authentication mode, which means that PostgreSQL server will not prompt for a password if the operating system user `postgres` is authenticated at the operating system level. It will trust the user assuming that the operating system has handled the authentication.
This behavior can be problematic if the server will be logged into by users other than the database administrators. If the other users are able to impersonate the system user `postgres`, then they can easily get superuser access to the PostgreSQL cluster. We can change this by disabling `peer` authentication in the `pg_hba.conf` file. However it is recommended to not allow non-DBA users, access the database server machine.

### Creating Users
You can use `psql` client utility to connect to the PostgreSQL cluster. You can create users and roles in the PostgreSQL server using the `psql` or `createuser` client tools. Run the below set of commands to create a regular user account (non superuser).
```bash
$ su - postgres
$ psql -c "CREATE ROLE newrole WITH LOGIN PASSWORD 'strongpassword';
```
OR
```bash
$ su - postgres
$ createuser --pwprompt --login newrole
```
Run the below set of commands to create a superuser account (like `postgres`).
```bash
$ su - postgres
$ psql -c "CREATE ROLE dbauser WITH LOGIN PASSWORD 'strongpassword' SUPERUSER;
```
OR
```bash
$ su - postgres
$ createuser --pwprompt --login --superuser newrole
```
> **Pro Tip:** If no password has been set up for a user, the stored password is null and password authentication will always fail for that user.

### Initialize phpPgAdmin
As already discussed above, you can optionally install the phpPgAdmin tool on the server system to access and administer the PostgreSQL cluster. 
```bash
$ sudo apt install phppgadmin
```
Upon installatoin of phpPgAdmin we need to edit the phpPgAdmin config file in the apache server to allow remote connections from clients outside the server host. Follow the below steps on the Debian server to modify the config file:
```bash
$ sudo nano /etc/apache2/conf-available/phppgadmin.conf
```
Locate the line that reads `Require local` and change it to `Require all granted`. Save the file and exit.
Restart the Apache service by running the below command.
```bash
$ sudo systemctl restart apache
```
Now, you will be able to connect to the PostgreSQL server and browse the objects using the URL: `http://<server_ip_address>/phppgadmin/` in the web browser of your choice.


[top](#top)
<div id="Configuring-PostgreSQL"/>

## Configuring PostgreSQL
### Editing postgresql.conf
The most fundamental way to set the parameters of PostgreSQL server is to edit the file postgresql.conf, which is normally kept in the data directory or the path is explicitly specified as a parameter at startup. In Debian, the configuration file is stored outside the data directory at the location: `/etc/postgresql/13/main/postgresql.conf`. A default copy is installed when the database cluster directory is initialized. If the file contains multiple entries for the same parameter, all but the last one are ignored.

All parameter names are case-insensitive. Every parameter takes a value of one of five types: boolean, string, integer, floating point, or enumerated (enum).  The type determines the syntax for setting the parameter:
|Value Type|Syntax|
|-|-|
|Boolean|`on`, `off`, `true`, `false`, `yes`, `no`, `1`, `0` (all case-insensitive)|	
|String|Value in single quotes, doubling any single quotes within the value. |
|Numeric|Integer and floating-point formats; fractional values are rounded to the nearest integer.|
|Numeric with unit|Some numeric parameters have an implicit unit, because they describe quantities of memory or time. Value written as string in single quotes. If a fractional value is specified with a unit, it will be rounded to a multiple of the next smaller unit if there is one. Valid memory units are `B` (bytes), `kB` (kilobytes), `MB` (megabytes), `GB` (gigabytes), and `TB` (terabytes). The multiplier for memory units is 1024, not 1000. Valid time units are `us` (microseconds), `ms` (milliseconds), `s` (seconds), `min` (minutes), `h` (hours), and `d` (days).|
|Enum|Enumerated-type parameters are written in the same way as string parameters, but are restricted to have one of a limited set of values. Case insensitive.|

>**Note:** The defalt unit of any setting can be found in the system catalog pg_settings.unit. 
>The list of acceptable enum values can be found in the system catalog pg_settings.enumvals. 

The configuration file is reread whenever the main server process receives a `SIGHUP` signal; this signal is most easily sent by running `pg_ctl` reload from the command line or by calling the SQL function `pg_reload_conf()`.

The system view [`pg_file_settings`](https://www.postgresql.org/docs/13/view-pg-file-settings.html "pg_file_settings") can be helpful for pre-testing changes to the configuration files, or for diagnosing problems if a SIGHUP signal did not have the desired effects.

### Include Config Files and Directories
The `postgresql.conf` file can also contain `include` or `include_dir` directives, which respectively specify a file or an entire directory of configuration files to include. These look like `include auditing.conf` or `include_dir 'conf.d'`. If the file name is not an absolute path, it is taken as relative to the directory containing the referencing configuration file. Inclusions can be nested. Non-absolute directory names are taken as relative to the directory containing the referencing configuration file. Within the specified directory, only non-directory files whose names end with the suffix `.conf` will be included.

### ALTER SYSTEM
In addition to `postgresql.conf`, a PostgreSQL data directory contains a file `postgresql.auto.conf`, which has the same format as `postgresql.conf` but is intended to be edited automatically, not manually. This file holds settings provided through the [`ALTER SYSTEM`](https://www.postgresql.org/docs/13/sql-altersystem.html "ALTER SYSTEM") command which is functionally equivalent to editing the `postgresql.conf`.

### ALTER DATABASE
The [`ALTER DATABASE`](https://www.postgresql.org/docs/13/sql-alterdatabase.html "ALTER DATABASE") command allows global settings to be overridden on a per-database basis.

### ALTER ROLE
The [`ALTER ROLE`](https://www.postgresql.org/docs/13/sql-alterrole.html "ALTER ROLE") command allows both global and per-database settings to be overridden with user-specific values.
>Values set with `ALTER DATABASE` and `ALTER ROLE` are applied only when starting a fresh database session. They override values obtained from the configuration files or server command line, and constitute defaults for the rest of the session.

### Session-Local settings
Once a client is connected to the database, PostgreSQL provides two additional SQL commands (and equivalent functions) to interact with session-local configuration settings:

|Command|Description|
|-|-|
|`SHOW <name>;`|Gets the current session-local setting by name|
|`SHOW ALL;`|Gets ALL the current session-local settings|
|`SELECT current_setting(name);`|Gets the current session-local setting by name|
|`SET configuration_parameter = value;`|Sets the current session-local setting by name|
|`SELECT set_config(setting_name, new_value, is_local_bool);`|Sets the current session-local setting by name|

### Updating pg_settings
The system view [`pg_settings`](https://www.postgresql.org/docs/13/view-pg-settings.html "pg_settings") can be used to view and change session-local values. Querying this view is similar to using SHOW ALL but provides more detail. It is also more flexible, since it's possible to specify filter conditions or join against other relations. Using UPDATE on this view, specifically updating the setting column, is the equivalent of issuing SET commands.

### Other Config files
Apart from the primary config file `postgresql.conf`, the PostgreSQL server also uses the below two config files to control authentication. The location of these files is provided in the `postgresql.conf` file.
|File|Description|
|-|-|
|`pg_hba.conf`|Specifies the configuration file for host-based authentication|
|`pg_ident.conf`|Specifies the configuration file for user name mapping|

### Changing location of Config files
By default, all three configuration files are stored in the database cluster's data directory. The parameters described in this section allow the configuration files to be placed elsewhere.
|Parameter|Description|
|-|-|
|`data_directory`|Specifies the directory to use for data storage. This parameter can only be set at server start.|
|`config_file`|Specifies the main server configuration file (customarily called `postgresql.conf`). This parameter can only be set on the `postgres` command line.|
|`hba_file`|Specifies the configuration file for host-based authentication (customarily called `pg_hba.conf`). This parameter can only be set at server start.|
|`ident_file`|Specifies the configuration file for user name mapping (customarily called `pg_ident.conf`). This parameter can only be set at server start.|
|`external_pid_file`|Specifies the name of an additional process-ID (PID) file that the server should create for use by server administration programs. This parameter can only be set at server start.|
In a default installation, none of the above parameters are set explicitly.If you wish to keep the configuration files elsewhere than the data directory, the `postgres` `-D` command-line option or `PGDATA` environment variable must point to the directory containing the configuration files, and the `data_directory` parameter must be set in `postgresql.conf` (or on the command line) to show where the data directory is actually located. Notice that `data_directory` overrides `-D` and `PGDATA` for the location of the data directory, but not for the location of the configuration files.

If you wish, you can specify the configuration file names and locations individually using the parameters `config_file`, `hba_file` and/or `ident_file`. `config_file` can only be specified on the `postgres` command line, but the others can be set within the main configuration file. If all three parameters plus `data_directory` are explicitly set, then it is not necessary to specify `-D` or `PGDATA`.


[top](#top)
<div id="Securing-PostgreSQL"/>

## Securing PostgreSQL Internals


#

### Securing data using Authentication
- Password Encryption - If SCRAM or MD5 encryption is used for client authentication, the unencrypted password is never even temporarily present on the server because the client encrypts it before being sent across the network.
- SSL Host Authentication - The client and server to provide SSL certificates to each other. Provides stronger verification of identity than the mere use of passwords. It also helps prevent “man in the middle” attacks.

### Securing Data at rest
- Encryption For Specific Columns - Using the pgcrypto module. The client supplies the decryption key and the data is decrypted on the server and then sent to the client.
- Data Partition Encryption - Storage encryption performed at the file system level or the block level. Linux file system encryption options include eCryptfs and EncFS, while FreeBSD uses PEFS.
- Client-Side Encryption - Data is encrypted on the client before being sent to the server, and database results have to be decrypted on the client before being used. Unencrypted data never appears on the database server and hence on the network.

### Securing Data in motion
- Encrypting Data Across A Network -  SSL-encrypted or GSSAPI-encrypted connections encrypt all data sent across the network. Stunnel or SSH can also be used to encrypt transmissions.
- Client-Side Encryption - Data is encrypted on the client before being sent to the server, and database results have to be decrypted on the client before being used. Unencrypted data never appears on the database server and hence on the network.


---
[top](#top)
<div id="PostgreSQL-Internals"/>

# PostgreSQL Internals
This section describes the internal working of PostgreSQL server.

[top](#top)
<div id="System-Catalogs"/>

## System Catalogs
- pg_database: List of all databases
- pg_namespace: 
- pg_class: 
- pg_tables:
- pg_index: 
- pg_roles:
- pg_user:
- pg_stat_activity: 
- pg_stat_user_indexes: To identify index usage stats. Also find out unused indexes.
- pg_authid - The password for each database user is stored in this system catalog.
- pg_file_settings
- pg_settings

[top](#top)
<div id="System-Functions"/>

### System Functions

pg_total_relation_size:
pg_size_pretty:

---

[top](#top)
<div id="PostgreSQL-Programming">

# PostgreSQL Programming

PostgreSQL is a relational database management system (RDBMS). Tables are grouped into databases, and a collection of databases managed by a single PostgreSQL server instance constitutes a database cluster.

### Identifiers and Key Words
- Key words and identifiers have the same lexical structure, meaning that one cannot know whether a token is an identifier or a key word without knowing the language.
#### Key Words
- Tokens such as `SELECT`, `UPDATE`, or `VALUES` are examples of key words, that is, words that have a fixed meaning in the SQL language.
### Identifiers
- They identify names of tables, columns, or other database objects, depending on the command they are used in. Therefore they are sometimes simply called “names”.
- SQL identifiers and key words must begin with a letter (a-z, but also letters with diacritical marks and non-Latin letters) or an underscore (_). Subsequent characters in an identifier or key word can be letters, underscores, digits (0-9), or dollar signs ($). Note that dollar signs are not allowed in identifiers according to the letter of the SQL standard, so their use might render applications less portable. The SQL standard will not define a key word that contains digits or starts or ends with an underscore, so identifiers of this form are safe against possible conflict with future extensions of the standard.
- The system uses no more than NAMEDATALEN-1 bytes of an identifier; longer names can be written in commands, but they will be truncated. By default, NAMEDATALEN is 64 so the maximum identifier length is 63 bytes. If this limit is problematic, it can be raised by changing the NAMEDATALEN constant in `src/include/pg_config_manual.h`.
- There is a second kind of identifier: the delimited identifier or quoted identifier. It is formed by enclosing an arbitrary sequence of characters in double-quotes ("). 
- Quoting an identifier also makes it case-sensitive, whereas unquoted names are always folded to lower case.

## Constants
### String Constants
- A string constant in SQL is an arbitrary sequence of characters bounded by single quotes ('), for example 'This is a string'. To include a single-quote character within a string constant, write two adjacent single quotes, e.g., 'Dianne''s horse'.
- Two string constants that are only separated by whitespace with at least one newline are concatenated and effectively treated as if the string had been written as one constant. For example:
	```sql
	SELECT 'foo'
	'bar';
	```
	is equivalent to:
	```sql
	SELECT 'foobar';
	```
### String Constants With C-Style Escapes
- PostgreSQL also accepts “escape” string constants, which are an extension to the SQL standard. An escape string constant is specified by writing the letter E (upper or lower case) just before the opening single quote, e.g., E'foo'. (When continuing an escape string constant across lines, write E only before the first opening quote.) Within an escape string, a backslash character (\) begins a C-like backslash escape sequence, in which the combination of backslash and following character(s) represent a special byte value.
- Any other character following a backslash is taken literally. Thus, to include a backslash character, write two backslashes (`\\`). Also, a single quote can be included in an escape string by writing `\'`, in addition to the normal way of `''`.
**\b**	- backspace
**\f**		- form feed
**\n**	- newline
**\r**		- carriage return
**\t**		- tab
**\o, \oo, \ooo (o = 0–7)**		- octal byte value
**\xh, \xhh (h = 0–9, A–F)**	- hexadecimal byte value
**\uxxxx, \Uxxxxxxxx (x = 0–9, A–F)**		- 16 or 32-bit hexadecimal Unicode character value

### String Constants With Unicode Escapes
- PostgreSQL also supports another type of escape syntax for strings that allows specifying arbitrary Unicode characters by code point. A Unicode escape string constant starts with U& (upper or lower case letter U followed by ampersand) immediately before the opening quote, without any spaces in between, for example `U&'foo'`. 
- Inside the quotes, Unicode characters can be specified in escaped form by writing a backslash followed by the four-digit hexadecimal code point number or alternatively a backslash followed by a plus sign followed by a six-digit hexadecimal code point number. For example, the string 'data' could be written as

	```sql
	U&'d\0061t\+000061'
	```
<br/>

> **Caution:** 
> If the configuration parameter  [standard_conforming_strings](https://www.postgresql.org/docs/13/runtime-config-compatible.html#GUC-STANDARD-CONFORMING-STRINGS)  is  `off`, then  PostgreSQL  recognizes backslash escapes in both regular and escape string constants. However, as of  PostgreSQL  9.1, the default is  `on`, meaning that backslash escapes are recognized only in escape string constants. This behavior is more standards-compliant, but might break applications which rely on the historical behavior, where backslash escapes were always recognized. As a workaround, you can set this parameter to  `off`, but it is better to migrate away from using backslash escapes. If you need to use a backslash escape to represent a special character, write the string constant with an  `E`.
In addition to  `standard_conforming_strings`, the configuration parameters  [escape_string_warning](https://www.postgresql.org/docs/13/runtime-config-compatible.html#GUC-ESCAPE-STRING-WARNING)  and  [backslash_quote](https://www.postgresql.org/docs/13/runtime-config-compatible.html#GUC-BACKSLASH-QUOTE)  govern treatment of backslashes in string constants.

<br/>

### Dollar-Quoted String Constants
- While the standard syntax for specifying string constants is usually convenient, it can be difficult to understand when the desired string contains many single quotes or backslashes, since each of those must be doubled. To allow more readable queries in such situations, PostgreSQL provides another way, called “dollar quoting”, to write string constants. A dollar-quoted string constant consists of a dollar sign ($), an optional “tag” of zero or more characters, another dollar sign, an arbitrary sequence of characters that makes up the string content, a dollar sign, the same tag that began this dollar quote, and a dollar sign. For example, here are two different ways to specify the string “Dianne's horse” using dollar quoting:
	```sql
	$$Dianne's horse$$
	$SomeTag$Dianne's horse$SomeTag$
	```
- Inside the dollar-quoted string, single quotes can be used without needing to be escaped. Indeed, no characters inside a dollar-quoted string are ever escaped: the string content is always written literally.

### Bit-String Constants
- it-string constants look like regular string constants with a B (upper or lower case) immediately before the opening quote (no intervening whitespace), e.g., B'1001'. The only characters allowed within bit-string constants are 0 and 1.
- Alternatively, bit-string constants can be specified in hexadecimal notation, using a leading X (upper or lower case), e.g., `X'1FF'`. This notation is equivalent to a bit-string constant with four binary digits for each hexadecimal digit.
- Both forms of bit-string constant can be continued across lines in the same way as regular string constants. Dollar quoting cannot be used in a bit-string constant.

### Numeric Constants
- Numeric constants are accepted in these general forms where digits is one or more decimal digits (0 through 9)
	```sql
	digits
	digits.[digits][e[+-]digits]
	[digits].digits[e[+-]digits]
	digitse[+-]digits
	```
-  At least one digit must be before or after the decimal point, if one is used. At least one digit must follow the exponent marker (e), if one is present. There cannot be any spaces or other characters embedded in the constant. Note that any leading plus or minus sign is not actually considered part of the constant; it is an operator applied to the constant.
- These are some examples of valid numeric constants:
	```
	42
	3.5
	4.
	.001
	5e2
	1.925e-3
	```
- A numeric constant that contains neither a decimal point nor an exponent is initially presumed to be type integer if its value fits in type integer (32 bits); otherwise it is presumed to be type bigint if its value fits in type bigint (64 bits); otherwise it is taken to be type numeric. Constants that contain decimal points and/or exponents are always initially presumed to be type numeric.
<br/>
<br/>

## Data Types and Precedence
<br/>
<br/>

## Operators
- An operator name is a sequence of up to NAMEDATALEN-1 (63 by default) characters from the following list:
	```sql
	+ - * / < > = ~ ! @ # % ^ & | ` ?
	```

### Operator Precedence (highest to lowest)

| Operator/Element              | Associativity | Description                                           |
|-------------------------------|---------------|-------------------------------------------------------|
| .                             | left          | table/column name separator                           |
| ::                            | left          | PostgreSQL-style typecast                             |
| [ ]                           | left          | array element selection                               |
| + -                           | right         | unary plus, unary minus                               |
| ^                             | left          | exponentiation                                        |
| * / %                         | left          | multiplication, division, modulo                      |
| + -                           | left          | addition, subtraction                                 |
| (any other operator)          | left          | all other native and user-defined operators           |
| BETWEEN IN LIKE ILIKE SIMILAR |               | range containment, set membership, string matching    |
| < > = <= >= <>                |               | comparison operators                                  |
| IS ISNULL NOTNULL             |               | IS TRUE, IS FALSE, IS NULL, IS DISTINCT FROM, etc     |
| NOT                           | right         | logical negation                                      |
| AND                           | left          | logical conjunction                                   |
| OR                            | left          | logical disjunction                                   |
### Special Characters
- Some characters that are not alphanumeric have a special meaning that is different from being an operator. Details on the usage can be found at the location where the respective syntax element is described. This section only exists to advise the existence and summarize the purposes of these characters.
- A dollar sign (`$`) followed by digits is used to represent a positional parameter in the body of a function definition or a prepared statement. In other contexts the dollar sign can be part of an identifier or a dollar-quoted string constant.
- Parentheses (`()`) have their usual meaning to group expressions and enforce precedence. In some cases parentheses are required as part of the fixed syntax of a particular SQL command.
- Brackets (`[]`) are used to select the elements of an array. See Section 8.15 for more information on arrays.
- Commas (`,`) are used in some syntactical constructs to separate the elements of a list.
- The semicolon (`;`) terminates an SQL command. It cannot appear anywhere within a command, except within a string constant or quoted identifier.
- The colon (`:`) is used to select “slices” from arrays. (See Section 8.15.) In certain SQL dialects (such as Embedded SQL), the colon is used to prefix variable names.
- The asterisk (`*`) is used in some contexts to denote all the fields of a table row or composite value. It also has a special meaning when used as the argument of an aggregate function, namely that the aggregate does not require any explicit parameter.
- The period (`.`) is used in numeric constants, and to separate schema, table, and column names.
<br/>
<br/>





### Table Characteristics
- Rows and Tuples are synonymous.
- Every Table in PostgreSQL has physical disk file(s).
```sql
CREATE TABLE foo (id int, name text);
SELECT relfilenode FROM pg_class WHERE relname = 'foo';

relfilenode
-------------
16384
```
- `relfilenode` is the table file name of that table, '16384' is the physical filename for table 'foo'.
- The physical files on the disk can be seen in the PostgreSQL `$PGDATA` directory.
- `$PGDATA` is the data directory of PostgreSQL and is setup during installation/startup.
- The files are stored in `$PGDATA` directory in the path `$PGDATA/base/<dat oid>/<relfilenode>`.
- Each file is further divided into 8k blocks.
- Tuples stored in a heap-table do not have any order.
- Select whole table, must be a sequential scan.
- Select table's rows ehere d = 5432, should not be a sequential scan.
- `ctid` is a hidden column in the result that returns the page number and tuple number of that record.
- For Example: (0, 1) means Page 0 (first page) and Tuple number 1 on that page, (3, 9) means Page 3 (fourth page) and Tuple number 9 on that page.
```sql
SELECT ctid, id, name FROM foo WHERE id = 5432;
  ctid  |  id  |  name  
--------+------+--------
 (0,1)  |   1  |  Alex
 (0,1)  |   2  |  Bob
(2 rows)
```
<br>
<br>

## PostgreSQL Indexes
- Indexes are used to identity entry points into tables.
- Index is used to locate the tuple(s) in the table.
- The sole reason to have an index is to improve query performance.
- Indexes are stored separately from the table's main storage (PostgreSQL Heap). Each index has its own file.
- Extra storage is required to store the index along with original table.
- PostgreSQL Locks the table when creating an index on it. Using the option `CONCURRENTLY` creates the index without locking the table.
- PostgreSQL planner will evaluate all available indexes and build an efficient execution plan.

### Methods of Indexing tables
1. **Single Column Index**
	- Index on a single column of a table.
	- Most common for small tables where lookup is expected to happen by a single column.
```sql
CREATE INDEX CONCURRENTLY id_idx ON foo USING BTREE (id);
CREATE INDEX test3_desc_index ON test3 (id DESC NULLS LAST);
```
2. **Multi-Column Index**
	- Composite Index on two or mode columns.
	- Up to 32 columns can be specified in a single index
	- The `WHERE` clause of the query needs to have an `AND` operator between filters on the columns indexed by the multicolumn indexes, else the index will not be used by the planner.
```sql
CREATE INDEX CONCURRENTLY id_name_dob_idx ON foo USING BTREE (id, name, dob);
```
3. **Expression Index**
	- Can create indexes on expressions
	- Not limited to B-Trees, but very helpful
	- Can fix performance issues with using functions in the `WHERE` clause of the queries.
	- Helps avoid using generated/computed columns in the base table.
```sql
CREATE INDEX CONCURRENTLY lowername_idx ON foo USING BTREE (lower(name));
CREATE INDEX CONCURRENTLY interval_idx ON bar USING BTREE ((dt + (INTERVAL '2 days')));
```
4. **Unique Indexes**
	- Indexes can also be used to enforce uniqueness of a column's value, or the uniqueness of the combined values of more than one column.
	- Currently, only B-tree indexes can be declared unique.
	- PostgreSQL automatically creates a unique index when a unique constraint or primary key is defined for a table. The index covers the columns that make up the primary key or unique constraint (a multicolumn index, if appropriate), and is the mechanism that enforces the constraint.
```sql
CREATE UNIQUE INDEX unique_empid_idx ON table (empid);
CREATE UNIQUE INDEX unique_email_loginname_idx ON table (email_address, login_name);
```
5. **Partial Index - Filtered Index**
	- A partial index is an index built over a subset of a table; the subset is defined by a conditional expression (called the predicate of the partial index).
	- Greatly reduces size of the index if the filter is highly selective.
	- Can be used to exclude values from the index that the typical query workload is not interested in.
	- Should not be used instead of table partitioning.
```sql
CREATE INDEX CONCURRENTLY id_partial ON bar USING BTREE (id) WHERE id < 10000;
```
6. **Covering Index - Include option**[^1]  [^2]
	- Improve index-only returns minimizing drawbacks
	- Store column values in leaf node tuples, but not in upper-level navigation entries.
	- Typically, the columns in the `SELECT` clause go in the `INCLUDE` clause.
```sql
CREATE INDEX empid_covering_idx ON table (empid) INCLUDE (name);
CREATE UNIQUE INDEX unique_empid_covering_idx ON table (empid) INCLUDE (name);
CREATE INDEX table_f_x ON table (f(x)) INCLUDE (x);
```
7.  **B-Tree tuple deduplication**[^3]
	- Special space efficient representation for duplicates when an optional technique is enabled.
	- A duplicate is a leaf level tuple where all indexed key columns have values that match values from atleast one other leave level tuple in the same index.
	- This is a lazy process and happens eventually, but can be forced using on `CREATE INDEX` and `REINDEX`.
	- It has some performance penalty for writes.
	- Can be disabled on the index level.
	- Saves a lot of space when B-Tree index is created on a column that has low cardinality.
	- Cannot be used with
		- character types with non-deterministic collations (case, accent)
		- numeric type
		- jsob type
		- float4/float8
		- container type
		- include indexes
```sql
CREATE INDEX id_amount_dedup ON orders USING BTREE (amount) WITH (deduplicate_items = ON);
CREATE INDEX id_amount_no_dedup ON orders USING BTREE (amount) WITH (deduplicate_items = OFF);
```
### Types of Indexes
1. **B-Tree Index (Balance Tree)**
	- This is the default index type in PostgreSQL. If the index type is not specified when creating index, B-Tree method is selected.
	- The B-Tree index structure stores the value of the indexed column along with the ctid of the tuple for lookup.
	- Only index type supporting the return of ordered output.
	- Only index type supporting uniqueness constraints.
	- Supports index-only scans and range scans. 
	- Range lookups are sorted in the index. Optimizer can read index for sorted results or read heap then sort.
	- Supports forward and backward scanning.
	- Supported Operators: `<`   `<=`   `=`   `>=`   `>`
	- Ideal for high cardinality data (columns with unique values)
	- Default fill-factor is 90.
```sql
CREATE INDEX CONCURRENTLY btree_idx ON foo USING BTREE (name);
CREATE INDEX CONCURRENTLY id_name_dob_idx ON foo USING BTREE (id, name, dob);
```
```sql
  ctid  |  name
--------+---------
  (0,1) |  Alex
  (0,2) |  Bob
```
2. **Hash Index**
	- Creates a hash-table of the column on which the index is created.
	- This index is used only for equality operations on the indexed column. It cannot do `> <` operations as the hash-value is a random string.
	- Supported Operators: `=`
	- No ordering and no Range lookups.
	- The Hash index structure stored the column value on which the index is created along with the ctid.
	- These indexes take extra space than the column itself if the hash value is larger in length than the indexed column. However if the column width is larger, then Hash index can actually save space as compared to B-tree.
	- No recommended before PostgreSQL v10 as they were not WAL-logged and not Replicated. Fixed since PG 10.
	- Vacuum on Hash indexes does not return space: frees up pages for future use.
	- Reindex or vacuum Full to reclaim space.
	- Hash index type cannot be the base for a Unique constraint (unlike B-Tree)
	- Default fill-factor is 75.
```sql
CREATE INDEX CONCURRENTLY btree_idx ON foo USING HASH (name);
```
3. **BRIN Index - Block Range Index**
	- Used when columns have a correlation with their physical location in the table.
	- Space optimized because BRIN index contains only three items:
		- Page Number
		- Min value of column
		- Max value of column
	- Splits the table into ranges of blocks, default range is 128 blocks. 
	- Works as an accelerator for Sequential Scan.
	- Supported Operators: `<`   `<=`   `=`   `>=`   `>`
	- The Hash index structure stored the column value on which the index is created along with the ctid.
	- If you Delete / Update a tuple in table, then this index needs to be rebuilt.
	- Ideal for insert only tables where date column is ever increasing.
	- In order to improve the insertion speed, blocks at the end of HEAP may NOT be updated to the BRIN index in real time.
	- From PG 10 onwards, when inserting blocks onto the next block range, vaccum is automatically triggered to count the BRIN stats of the previous block range.
```sql
CREATE INDEX CONCURRENTLY btree_idx ON foo USING BRIN (name);
```
4. **GIN Index - Generalized inverted index**
```sql
CREATE INDEX CONCURRENTLY gin_idx ON bar USING GIN (json_data);
```
- GIN is to handle where we need to index composite values.
- Used for documents and arrays, JSON etc.
- Slow while creating the index because it needs to scan the document upfront.
- Stored values of the JSON data
5. **GiST- Generalized Search Tree**
```sql
CREATE INDEX CONCURRENTLY gin_idx ON bar USING GIST (json_data);
```
- Tree structure access method
- Used to find the point within a box
- Used for Full Text Search and spatial data.
6. **SP-GiST**
```sql
CREATE INDEX CONCURRENTLY gin_idx ON bar USING GIST (json_data);
```


<br/>
<br/>

[^1]: Introduced in PostgreSQL v11 
[^2]: Introduced in PostgreSQL v12
[^3]: Introduced in PostgreSQL v13


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAxMTQ3NzczLDE4MzExOTQ1NTUsLTE4OT
QyMTAwMCwtODA4MzM3NDAzLDE4Njg4NDEyNjYsLTE2NDcxMzQ5
NTYsLTMyMTgyNzQ0MCwtMTMxODkwMTcxMSwxMTU1MDAzNDI2LC
0xMjk0NTE2ODcwLC0xNzI1OTMyOTY0LDExNjQ2MjY0NDksLTE5
NjkxOTA2NjcsMTkzODI5NDE5MiwtMTcxMDg2NDk3MywtMjExMj
E1MzE0MywtMTUzNTM3NzU4MiwxNDkyNjgzOTMsMjA2NDk1MjI1
OSw0ODI0MDQwOTFdfQ==
-->