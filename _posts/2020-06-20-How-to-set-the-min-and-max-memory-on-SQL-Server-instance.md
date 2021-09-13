---
layout: post
title: How to set the min and max memory on SQL Server instance?
date: {}
summary: Learn how to configure memory in the SQL Server.
categories: Configuration Tuning
published: true
---

Setting appropriate values for the min. and max. memory on a SQL Server instance is important for its performance. In an ideal world, SQL Server need not share the server resources with any other application. However, for many shops, this is not the case as the VM running SQL Server could be assigned several roles. Hence it is important to measure the memory requirements of all the programs running on the Server and set the SQL Server’s memory accordingly.

## Setting memory using SQL Server Management Studio

* Inside SSMS, Right Click on the Instance and select “Properties”.
    
![SSMS](/img/posts/2020-06-20-How-to-set-the-min-and-max-memory-on-SQL-Server-instance/01.png){: height="100%" }

* In the Properties window, choose “Memory” and you will now see options to set the Minimum and Maximum Server memory in MB.

![Properties](/img/posts/2020-06-20-How-to-set-the-min-and-max-memory-on-SQL-Server-instance/02.png)

* Here you can set the desired values for the Min and Max Server memory in MB. 

> Please note that you have to be extra cautious when setting these parameters as fat-fingering an incorrect value can bring down your SQL Server instance and will be very painful to fix. I have been there and done that and its not a very pleasant experience.



## Setting memory using T-SQL script

* Alternatively, you can also set the Min and Max Server memory in MB by running the following T-SQL code.

<script src="https://gist.github.com/relationaldba/398835984cd57e55a0ff31f98a149ac8.js"></script>



## Consider these points when setting the memory

Here are some considerations when setting the Min and Max memory for SQL Server:

1. Is your Box dedicated to SQL Server?
If the answer to this question is Yes, then, you want to allocate as much memory to the SQL Instance as possible. I would leave around 30% of the Server memory for the Windows OS and other utilities like Antivirus, RDP sessions of the users, SSMS etc and allocate around 70% to SQL Server as the max server memory.

2. Are you running multiple instances of SQL Server on your box?
If the answer to this question is Yes, then you want to distribute the memory between these instances based on the total database size and workload so that they do not fight among each other for RAM. Example Instance1 gets 40% RAM, Instance2 gets 30% RAM and the remaining 30% we leave for the OS and other utilities.

3. What’s the recommended value for Minimum memory?
The out of box setting for Minimum server memory for SQL Server is 0 MB. This means SQL will not reserve any fixed memory if it experiences memory pressure. This setting, in my opinion, should not be changed unless you are fixing issues with Memory pressure on the SQL Instance caused by other utilities running on the box. Setting this value to a non-zero value could, in some cases, cause issues with memory management of Windows and the Instance could crash due to excessive memory pressure.

## Final Thoughts

In most cases, you would be fine setting the Maximum memory to 70% of the total server memory. You don’t have to set the Minimum memory unless you have a very good reason to do so, and even in such cases try to fix the external sources that cause memory pressure on SQL server rather than setting a non-zero value for Minimum memory.

I hope you found this article helpful.
