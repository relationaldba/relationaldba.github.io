---
published: true
title: How to Shrink Databases and Files?
date: 2020-04-02T00:00:00.000Z
layout: post
summary: Learn how to shrink the databases and files efficiently.
categories: SQLServer Administration
---
![Background](/img/posts/2020-04-02-How-to-Shrink-Databases-and-Files/bg.png)
**NO, it does NOT sound like a plan!!**

Shrinking Databases and Files is largely considered “Taboo” in the DBA community, and for good reasons. One – Shrinking the database causes pages at the end of the file to be moved to the beginning of the file leading to fragmentation of indexes, Two – In most cases, shrinking will not help as the database will again grow the size before you shrank it, as whatever caused it to grow initially will happen again (given enough time) and will render shrinking useless, and Three – Shrinking will stress CPU, Memory, and I/O and could significantly impact the server’s performance.

However, if you must shrink the database, let me show you the least evasive way.

## Should I shrink the database?
Here are some of the reasons why you may consider shrinking the database.

1. After truncating huge tables as a part of a cleanup task.
2. Moving old data to a different FG or a different database for archival purposes.
3. Log file grown out of proportion due to huge index rebuild operation or a huge DML operation.
4. Free space created due to dropping several unused indexes or tables.
5. When you don't have an option to expand the storage and need to free up space for other databases to grow.
6. When you restore a database with lot of free space from a backup for read-only purposes.

I am guilty of shrinking databases due to all of the above reasons. However, shrinking databases has always been the last option to claim space. I would almost always prefer to expand the storage, given that the storage costs have been drastically reduced. For example, 100GB of EBS SSD drive on AWS would cost around $10 per month. Other cloud vendors also have a similar cost. It mostly boils down to convincing your manager or the sysadmins of the long-term pain that you would avoid by investing a small amount in storage.

## Before you Shrink
Determine how much space savings are you going to get by shrinking the database or its files. You can review my article <a rel="noreferrer noopener" href="https://relationaldba.com/index.php/2020/03/17/check-free-space-in-database-and-log-files/" target="_blank">Check Free Space in Database and Log files</a> to check the free space in files and decide if it's worth going through the pain of shrinking the database or its files. Typically I would not encourage you to shrink the database if the space savings is less than the size of your biggest table or index.

I would also measure the fragmentation of the indexes before I run the shrink operation on the database. This helps me understand the amount of fragmentation introduced by the shrink operation.

## Never run DBCC SHRINKDATABASE
DBCC SHRINKDATABASE is like an all-out assault on all the files and filegroups of the database. The DBCC SHRINKDATABASE command will run the shrink operation on all the files including the log files of the database and reclaim space. This is certainly not what you want to do on your production database. Instead, you want to pick the file that will give you the most bang for your buck, and only shrink that file.
<script src="https://gist.github.com/relationaldba/767784ccdd4fc56055247e72fa10f0e5.js"></script>
The DBCC SHRINKDATABASE command accepts a target_percentage as the second argument. It will try to shrink all the files of the database to achieve free space close to the target percentage. Is the percentage of free space that you want to be left in the database file after the database has been shrunk.

## Instead run DBCC SHRINKFILE
DBCC SHRINKFILE is a precise surgical strike on the file that you need to shrink. This is especially helpful when your database has several files and you wish to shrink only one file at a time.
<script src="https://gist.github.com/relationaldba/5a6e427143dd91596ba56e0624b6ae77.js"></script>
The DBCC SHRINKFILE command accepts a target_size as the second argument. The target_size argument is the file's desired size in MBs. If not specified, DBCC SHRINKFILE will shrink to the file creation size specified at the time of creating the database.

## Locks held during shrinking
Just like any other operation in SQL Server, DBCC SHRINKDATABASE and DBCC SHRINKFILE hold locks on the database objects while performing the shrink operation. This could impact the overall performance of the applications if your database is very busy. Shrinking will block other processes that need locks on the objects that are locked by the DBCC SHRINK command.
Here is how the locks look like in my demo environment. This is an output of Adam Machanic's sp_whoisactive stored procedure run with the parameter @get_locks = 1. We can clearly see that there are eXclusive locks on the Database File and the Pages of the object it is trying to shrink. I would hence perform all shrink operations during non-peak hours if not during a planned maintenance window.
<script src="https://gist.github.com/relationaldba/7164641a6198fd507c9218afa7acdf40.js"></script>

## Shrink the files in small chunks
One of the tricks to shrink a huge database example 1 TB database with 500 GB free space is to break down the shrink operation into smaller units of work. I would normally split the shrink operation over several days to keep the maintenance window as low as possible. For example, I would normally shrink the size by only 5 or 10 GB at a time and keep repeating the operation until I clear out the entire 500 GB of free space.
Here is how the database could be shrunk by 50 GB every day.
<script src="https://gist.github.com/relationaldba/18765c23100575f4f0386fd4abc0f416.js"></script>

## TRUNCATEONLY option
This is one of the options that I strongly recommend if you want to quickly shrink files without any impact on your indexes and page movement. This option essentially de-allocates the whitespace (empty space) at the end of the file. Because it does not re-order pages, it is very quick and efficient and does not cause any fragmentation. However, if there is not enough whitespace at the end of the file, then you may not see any reduction in the file size. TRUNCATEONLY option will ignore the empty space elsewhere in the data file.
<script src="https://gist.github.com/relationaldba/3ed67508606102cdf7ee0e06a78cc188.js"></script>

## What to do after shrinking

1. Always measure the fragmentation of the indexes in your database before and after the shrink operation. It will help you gauge the damage inflicted by the shrink operation on the fragmentation of your indexes.
2. I would normally run the reorganize index command to reduce the fragmentation right after the shrink completes.
3. I also would confirm that there is enough space available on the disks for the database to grow again.
4. If the database is expected to grow at a faster rate, I would ensure the filegrowth value is set to an optimal value to avoid repetitive growth in small chunks as it could lead to file defragmentation.
5. In some cases I would consider enabling <a href="https://docs.microsoft.com/en-us/sql/relational-databases/databases/database-instant-file-initialization" target="_blank" rel="noreferrer noopener">Instant File Initialization</a> to help SQL Server grow files faster without having to zero-writing the newly allocated chunk of space. Use this feature with caution as it has some security issues.

## Final thoughts

Shrinking is not as bad as it sounds if done with proper analysis. Explore other options to fix the space issue before you settle for shrinking. Money spent on expanding the storage by a few hundred gigs is money well spent. Plan the downtime for shrink operation in advance as it could lead to blocking and unhappy users. I would recommend reading Brent Ozar's great blog on <a rel="noreferrer noopener" href="https://www.brentozar.com/archive/2017/12/whats-bad-shrinking-databases-dbcc-shrinkdatabase/" target="_blank">What’s So Bad About Shrinking Databases</a>. Its a great demo of how shrinking impacts the indexes and what you can expect after rebuilding indexes.

Hope you found this article helpful!
