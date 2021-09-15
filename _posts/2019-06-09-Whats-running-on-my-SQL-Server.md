---
layout: post
title: What’s running on my SQL Server?
date: 2019-06-09T00:00:00.000Z
summary: Find out currently the running queries on SQL Server.
categories: SQLServer TSQL Scripts
published: true
---
![Background](/img/posts/2019-06-09-Whats-running-on-my-SQL-Server/01-Whats-up-SQL-Server-1024x512.png)

Database Administrators and Developers often want to know what’s currently running on the SQL Server. SQL server has some great system stored procedures and Dynamic Management Views that are useful in finding out the currently running queries, their state, and whether queries are blocked or waiting on some resource.

In the below article, let me show you the scripts that I use in my day-to-day work as a Database Administrator to find what’s going on the SQL Server.

## sp_who / sp_who2

sp_who and sp_who2 are system stored procedures in the master database. These are shipped by Microsoft in all editions of SQL Server. Although, sp_who2 is not officially documented on Microsoft Docs, it returns the same output as sp_who. Both these stored procedures return the basic information of users, sessions, and processes running on the SQL Instance.

I usually use DBCC INPUTBUFFER (SPID) to get the SQL text of a particular SPID from the output of sp_who2.

<script src="https://gist.github.com/relationaldba/15f30b87eb653588f5d4e4a6fe4e06d3.js"></script>

Here is how the output looks like:

![sp_who2](/img/posts/2019-06-09-Whats-running-on-my-SQL-Server/02-sp_who2-1024x313.png)

These SPs are great at showing how many processes are currently active on SQL Server, blocking, logins and how much CPU and DiskIO each SPID has consumed. You can also pass the login name parameter to this procedure to filter the output for a specific login.

sp_who2 is not my first choice when finding out what’s currently running on Server, although, it can be pretty handy when you want to take a quick peek at the current processes and when you don’t have your other monitoring tools at hand.

Read more at https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-who-transact-sql

## Microsoft’s Activity Monitor Query

So when you fire up the Activity monitor on SQL Server Management Studio, and expand Processes section, SQL Server runs this query in the background. Wait a second, you asking – How do I know this? Let me tell you how. On your SQL box, start a trace using SQL Server Profiler and select SQL:BatchCompleted events. Then fire up the Activity Monitor on SSMS and expand Processes section. Immediately you will notice the the trace will capture the below query 🙂

This query is very handy in finding out the currently executing SQL queries and their state. If you plan on using this, I would recommend you add a where clause to filter system SPIDs (spid <= 50) to view only the user sessions.

<script src="https://gist.github.com/relationaldba/1d02194975e013c9b7c1d1e45d966e66.js"></script>

Here’s how a typical output of this query looks like.

![Activity Monitor](/img/posts/2019-06-09-Whats-running-on-my-SQL-Server/03-Activity-Monitor-1024x345.png)

## Adam Machanic’s sp_WhoIsActive

This is a great stored procedure and can be downloaded for free at http://whoisactive.com. I have used sp_WhoIsActive in the past and I found it to be quite helpful. It has a slight learning curve if you want to use the parameters to modify the output or zoom into a specific problem. The reason I do not use it anymore is that my production servers cannot have code that is not developed inhouse by the organization that I work for. So, I had to look for an alternative that could be just as helpful like sp_WhoIsActive.

Once you download the SP from http://whoisactive.com you can install it in the master database, that way you can call it while you are in context of any user database.

```sql
EXEC sp_whoisactive;
```

## The query that I use

I wrote the below query for my personal use, and would encourage you all to try this in your environment. This query is helpful in many ways. Not only does it show the running queries but also shows information on Blocking, Wait Statistics, CPU and IO consumption, Memory Grants and Execution plans as well. Plus, I don’t have to worry about installing an SP into my production machines. I have personally found this query to be pretty nifty in my day-to-day monitoring and troubleshooting.

> This works on SQL 2016 and later only.

<script src="https://gist.github.com/relationaldba/39754423e5d57aac52563e9ccba5aaab.js"></script>

Here’s how a typical output of this query looks like.

![RelationalDBA's Query](/img/posts/2019-06-09-Whats-running-on-my-SQL-Server/04-RelationalDBAs-Query-1024x243.png)

Final Thoughts

I almost always default to using the <a href="https://relationaldba.com/index.php/2019/06/09/whats-running-on-my-sql-server/#RelationalDBAActivityMonitorQuery" rel="noreferrer noopener" target="_blank">Query that I wrote above</a>, mostly because I can make quick edits to the script on runtime and change the output the way I like. It gives me the ability to format the query output the way I want without having to remember the parameters to a stored procedure. But that’s just me.

I hope you found this article helpful.