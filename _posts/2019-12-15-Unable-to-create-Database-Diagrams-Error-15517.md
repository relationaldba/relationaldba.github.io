---
published: true
title: "Unable to create Database Diagrams. Error: 15517 – Cannot execute as the database principal because the principal “dbo” does not exist…"
date: 2019-12-15T00:00:00.000Z
layout: post
summary: "Learn how to fix the error 15517 – Cannot execute as the database principal..."
categories: Troubleshooting Error
---
![Background](/img/bg-mssql-1024x288.png)

While creating a database diagram the other day, I ran into the below error on SQL Server Management Studio.

```
Cannot execute as the database principal because the principal “dbo” does not exist, this type of principal cannot be impersonated, or you do not have permission. (Microsoft SQL Server, Error: 15517)
For help, click: http://go.microsoft.com/fwlink?ProdName=Microsoft%20SQL
%20Server&ProdVer=15.00.2070&EvtSrc=MSSQLServer&EvtID=15517&LinkId=20476
```

## How to re-produce the error?

As you can see in the screenshot below, I expanded the database node in the object explorer and right-clicked on the database diagram. In the shortcut menu, I selected “New Database Diagram” and immediately got the error shown below. This error did not go away even if I chose to “Install Diagram Support“.

![Database Diagram Menu in SSMS](/img/posts/2019-12-15-Unable-to-create-Database-Diagrams-Error-15517/01-New-Database-Diagram.png)

![Error Message](/img/posts/2019-12-15-Unable-to-create-Database-Diagrams-Error-15517/02-Cannot-execute-because-the-database-principal-dbo.png)


## How to fix this error?

I tried creating a database diagram for another database on the same SQL Instance and it worked perfectly well. It immediately struck me that this error could be because the AdventureWorks2019 database that I wanted to create a database diagram for, is restored from a backup provided by Microsoft.

I know from experience that if a database is restored on a different instance of SQL Server than where it was backed up, you will almost always end up having orphan users in that database. This includes the database owner. The owner field of the database will be empty if the database was originally backed up on another server. Orphan Users are the database principals that are not linked to any SQL Server Logins. The relationship between a user on a database and the login on the instance is defined by the SID (security identifiers) of those principals. In order to be paired, both the database principal and the SQL Server Instance principal need to have the same SID.

So it was quite obvious that the SID of the dbo user on the database did not match with the sysadmin login on my SQL Server instance where I restored it.

I verified this by going to the properties of the “dbo” user in the AdventureWorks2019 database. The login name field was blank, thus confirming my hypothesis.

![Orphan User dbo](/img/posts/2019-12-15-Unable-to-create-Database-Diagrams-Error-15517/03-Orphan-user-dbo.png)

This can also be verified using the below T-SQL Script:

<script src="https://gist.github.com/relationaldba/2c3efa0b19cd7d9bced52bb56a0701a6.js"></script>

The fix for this issue is pretty simple. Microsoft has provided us a system Stored Procedure that can fix orphan users. In the background the SP updates the SID of the “dbo” user in the database to the SID of the Database Owner login.

<script src="https://gist.github.com/relationaldba/01f5d484868de1fba3f74f98a5719ae7.js"></script>

Database diagrams worked perfectly well after executing this script. Yayy..!!

## Final Thoughts

I don’t strongly feel that this problem can be classified as a bug in SQL Server, however I do strongly feel that Microsoft should take care of the orphaned “dbo” user automatically at the time of Database restore (or atleast provide us an option in the RESTORE DATABASE command itself).

I hope you found this article helpful.