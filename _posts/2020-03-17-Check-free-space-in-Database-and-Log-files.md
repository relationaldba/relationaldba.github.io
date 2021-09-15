---
published: true
title: Check free space in Database and Log files
date: 2020-03-17T00:00:00.000Z
layout: post
summary: Do your databases have a lot of free space?
categories: SQLServer Administration
---
![Background](/img/love-sql-server.png)
SQL Server stores data in the files and file groups. The smallest, most atomic chunk of data is a SQL Server page that has a size of 8KB. Data and log files are made up by a collections of such 8KB pages. The storage engine allocates new pages to the files as they grow. Each time the file grows, the storage engine requests the operating system to allocate space for the file to physically grow. The amount of space it requests is determined by the auto-growth setting of that file.

On the other hand, SQL Server de-allocates the pages when they become empty due to deletion, truncation, log backup or compacting the data. However, SQL Server will not release the empty space (unused pages) that gets created because of page de-allocation, back to the Operating System. It will continue to hold ownership to that physical free space inside the file, until the file or the database is forced to give up that space by a **DBCC SHRINK** operation. 

## Check Free space using SSMS

You can check the free space inside the database by going to the database properties in SSMS (Right Click on Database Name in Object Explorer > Click Properties) as shown in the screenshot below. This way you get the total free space inside all the files combined in the database. It does not show you the free space in each of the files individually.

![Free Space using SSMS](/img/posts/2020-03-17-Check-free-space-in-Database-and-Log-files/free-space-using-ssms.png)

## Check Free space using T-SQL Script

Alternatively, I like to check for the free space inside all the files individually by running the script below. You need to be connected to the database for which you want to check the free space.

<script src="https://gist.github.com/relationaldba/05ae6c7bcea3d07bf81efaff24b65199.js"></script>

The output of this query looks like this:

![Free Space using TSQL](/img/posts/2020-03-17-Check-free-space-in-Database-and-Log-files/free-space-using-tsql.png)

## Final Thoughts

I like to keep a track of free space inside all the databases on my production box, however I do not shrink any files unless I have run out of all the alternatives. DBAs should regularly collect information like Free Space, Auto-Growth size and File Size of all the database in their Production Environment. It helps with predicting growth of the databases, resize storage before it runs out if space and we can do some cool stuff like plotting a graph of the size over time.

I hope this helps.