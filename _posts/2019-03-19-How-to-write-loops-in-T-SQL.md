---
layout: post
title: How to write loops using T-SQL in SQL Server?
date: 2019-03-19T00:00:00.000Z
summary: Learn how to loop using T-SQL.
categories: SQLServer TSQL Scripts
published: true
---
![Background](/img/posts/2019-03-19-How-to-write-loops-in-T-SQL/01-Ooo...-I-hate-these-fruit-loops-1024x512.png)

Loops are the fundamental building blocks of several programming languages. Many real world problems can be solved by iterating the logic using loops. However, loops aren’t that popular among the SQL community. The reason is because loops inherently dictate a row by row operation instead of a set based operation. SQL Server is painfully slow while performing row based operations; it is optimized to perform set based operations instead.

Having said that, often times you may run into situations where you have to use loops in SQL.

Let me show you the ways in which I write loops in SQL Server.

## The “GO n” Loop

The GO keyword is a batch separator in SQL Server. It is also used to scope the life and accessibility of temp objects like variables. However a lesser known fact about the GO keyword is that it accepts a number right after it, and the number acts as an instruction to SQL to execute the batch that many number of times. For example if you need to run a piece of code 10 times, you simply need to write GO 10 and execute it. SQL Server understands that it is expected to run the batch 10 times.

<script src="https://gist.github.com/relationaldba/3087840899cb3a3b93254b00f01cae22.js"></script>

## The GOTO Loop

The GOTO keyword does not inherently loop or iterate the code but gives you the power to control the flow of your code. This is similar to one of those control-flow keywords used in procedural languages like C and BASIC. You can direct SQL to GOTO a particular line of code that is denoted by a SQL label.

A label in SQL is just a unique name assigned to a line in the SQL Code. You can have SQL Server to transfer the execution flow to a particular label using the GOTO label command. I use this method to batch delete rows in chunks of a couple thousand at a time from a huge table and iterate through the loop until all the required rows are deleted. This helps me avoid locking the entire table and I don’t block other users for a long period of time. Other queries to execute between the loops.

<script src="https://gist.github.com/relationaldba/108c92ae1c468784589cb7642599ae25.js"></script>

## The WHILE Loop

The WHILE loop is very powerful when you know the number of iterations before hand. You would normally count the loop executions manually using a counter and decide a condition or a counter value when the loop exits. Here is how you can write a simple WHILE loop.

<script src="https://gist.github.com/relationaldba/8cd4305976c110f7b612109b29ccba39.js"></script>

## The CURSOR Loop

Cursors are the most sophisticated way of iterating a block of code in T-SQL. Cursors take a result set of a query as an input and then iterate over each row one by one. I have found CURSORs to be life saviors in situations where I needed to perform some heavy-duty operations on a row-by-row basis. CURSORs can go forward and backward and that makes it a very real-word solution for row based operations.

I strongly recommend reading <a href="https://docs.microsoft.com/en-us/sql/t-sql/language-elements/declare-cursor-transact-sql?view=sql-server-ver15" rel="noreferrer noopener" target="_blank">Microsoft’s documentation</a> related to CURSORs to understand all the options that it packs. Here is a sample form of a CURSOR in T-SQL.

<script src="https://gist.github.com/relationaldba/55faf24e232ae3acdbfd3f1f00343d1d.js"></script>

## Final Thoughts

I do not recommend using these loop techniques before first considering set based alternatives. However, if you find yourself in a position where looping is the only viable solution, go for the one that serves your purpose.

I hope you found this article helpful.