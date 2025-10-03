https://www.sqlshack.com/when-to-use-temporary-tables-vs-table-variables/

# When to Use SQL Temp Tables vs. Table Variables

It is very beneficial to store data in SQL Server temp tables rather than manipulate or work with permanent tables. Let’s say you want full DDL or DML access to a table, but don’t have it. You can use your existing read access to pull the data into a SQL Server temporary table and make adjustments from there. Or you don’t have permissions to create a table in the existing database, you can create a SQL Server temp table that you can manipulate. Finally, you might be in a situation where you need the data to be visible only in the current session.

SQL Server supports a few types of SQL Server temp tables that can be very helpful.

Before we proceed, if you want to follow along with any code samples, I suggest opening SQL Server Management Studio:

![](https://www.sqlshack.com/wp-content/uploads/2017/02/word-image-80.png)

## Local SQL temp tables

Local SQL Server temp tables are created using the pound symbol or “hashtag” followed by the table name. For example: #Table_name. SQL temp tables are created in the tempdb database. A local SQL Server temp table is only visible to the current session. It cannot be seen or used by processes or queries outside of the session it is declared in.

Here’s a quick example of taking a result set and putting it into a SQL Server temp table.

|   |   |
|---|---|
||/*Insert Databases names into SQL Temp Table*/<br><br>BEGIN TRY<br><br>DROP TABLE #DBRecovery<br><br>END TRY<br><br>BEGIN CATCH SELECT 1 END CATCH<br><br>SELECT ROWNUM = ROW_NUMBER() OVER (ORDER BY sys.[databases]),<br><br>    DBName = [name],<br><br>    RecoveryModel = [recovery_model_desc]<br><br>INTO #DBRecovery<br><br>FROM sys.[databases]<br><br>WHERE [recovery_model_desc] NOT IN ('Simple')|

One of the most often used scenarios for SQL Server temp tables is within a loop of some sort. For example, you want to process data for a SQL statement and it you need a place to store items for your loop to read through. It provides a quick and efficient means to do so. See the code sample above, your loop can now reference the SQL Server temp table and process the records that meet the criteria of your goal.

Another reason to use SQL Server temp tables is you have some demanding processing to do in your sql statement. Let’s say that you create a join, and every time you need to pull records from that result set it has to process this join all over again. Why not just process this result set once and throw the records into a SQL temp table? Then you can have the rest of the sql statement refer to the SQL temp table name. Not only does this save on expensive query processing, but it may even make your code look a little cleaner.

There is one point that I want to make however. If the session that we’re working in has subsequent nested sessions, the SQL Server temp tables will be visible in sessions lower in the hierarchy, but not above in the hierarchy. Please allow me to visualize this.

In this quick diagram, a SQL temp table is created in Session 2. The sessions below it (sessions 3 and session 4) are able to see the SQL Server temp table. But Session 1, which is above session 2, will not be able to see the SQL Server temp table.

![](https://www.sqlshack.com/wp-content/uploads/2017/02/word-image-81.png)

The SQL temp table is dropped or destroyed once the session disconnects. Many times you’ll see developers use the “DROP #Table_Name” command at the end of their statement just to clean up. But it is entirely up to you and what you’re trying to accomplish.

Also note, that in the event of name conflict (remember that SQL Server temp tables are created in the tempdb) SQL server will append a suffix to the end of the table name so that it is unique within the tempdb database. But this process is transparent to the developer/user. You can use the same name that you declared as it’s confined to that session.

## Global SQL temp tables

Global SQL temp tables are useful when you want you want the result set visible to all other sessions. No need to setup permissions. Anyone can insert values, modify, or retrieve records from the table. Also note that anyone can DROP the table. Like Local SQL Server temp tables, they are dropped once the session disconnects and there are no longer any more references to the table. You can always use the “DROP” command to clean it up manually. Which is something that I would recommend.

![](https://www.sqlshack.com/wp-content/uploads/2017/02/word-image-82.png)

To create a global SQL temp table, you simply use two pound symbols in front of the table name. Example: ##Global_Table_Name.

## Table Variables

Table variables are created like any other variable, using the DECLARE statement. Many believe that table variables exist only in memory, but that is simply not true. They reside in the tempdb database much like local SQL Server temp tables. Also like local SQL temp tables, table variables are accessible only within the session that created them. However, unlike SQL temp tables the table variable is only accessible within the current batch. They are not visible outside of the batch, meaning the concept of session hierarchy can be somewhat ignored.

As far as performance is concerned table variables are useful with small amounts of data (like only a few rows). Otherwise a SQL Server temp table is useful when sifting through large amounts of data. So for most scripts you will most likely see the use of a SQL Server temp table as opposed to a table variable. Not to say that one is more useful than the other, it’s just you have to choose the right tool for the job.

Here is a quick example of setting up and using a table variable.

|   |   |
|---|---|
||DECLARE @TotalProduct AS TABLE<br><br>(ProductID INT NOT NULL PRIMARY KEY,<br><br>Quantity INT NOT NULL)<br><br>INSERT INTO @TotalProduct<br><br>         ( [ProductID], [Quantity] )<br><br>SELECT<br><br>  A.[ProductID],<br><br>  [Quantity] = SUM(B.Quantity)<br><br>FROM dbo.Product AS A<br><br>INNER JOIN dbo.SalesDetails AS B ON A.ProducitID = B.ProductID|

We’ve created a table variable that will hold information regarding total quantities of a certain product sold. This is a very simplified example, and we wouldn’t use it if it contained a lot of rows. But if we were only looking at a few products this could really well. Once the table variable is populated you can then join this as a table to yet another table and gather whatever information you need. So there is a lot of flexibility and allows the developer to be quite creative.

Also, on a final note, in terms of transactions on table variables. If a developer rolls back a transaction which includes changes to the table variables, the changes made to the table variables within this particular transaction will remain intact. That is to say, other parts of this transaction in question will be rolled back, but anything referencing the table variable will not, unless that portion of your script is in error.

## See more

To boost SQL coding productivity, check out these [SQL tools](https://www.apexsql.com/prices.aspx?utm_source=sqlshack&utm_campaign=free&utm_medium=native_link&utm_content=sql-shack) for SSMS and Visual Studio including T-SQL formatting, refactoring, auto-complete, text and data search, snippets and auto-replacements, SQL code and object comparison, multi-db script comparison, object decryption and more