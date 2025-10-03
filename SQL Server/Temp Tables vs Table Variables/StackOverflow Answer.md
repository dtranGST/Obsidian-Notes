https://stackoverflow.com/a/64891

There are a few differences between Temporary Tables (#tmp) and Table Variables (@tmp), although using tempdb isn't one of them, as spelt out in the MSDN link below.

As a rule of thumb, for small to medium volumes of data and simple usage scenarios you should use table variables. (This is an overly broad guideline with of course lots of exceptions - see below and following articles.)

Some points to consider when choosing between them:

- Temporary Tables are real tables so you can do things like CREATE INDEXes, etc. If you have large amounts of data for which accessing by index will be faster then temporary tables are a good option.
    
- Table variables can have indexes by using PRIMARY KEY or UNIQUE constraints. (If you want a non-unique index just include the primary key column as the last column in the unique constraint. If you don't have a unique column, you can use an identity column.) [SQL 2014 has non-unique indexes too](https://stackoverflow.com/questions/886050/sql-server-creating-an-index-on-a-table-variable/17385085#17385085).
    
- Table variables don't participate in transactions and `SELECT`s are implicitly with `NOLOCK`. The transaction behaviour can be very helpful, for instance if you want to ROLLBACK midway through a procedure then table variables populated during that transaction will still be populated!
    
- Temp tables might result in stored procedures being recompiled, perhaps often. Table variables will not.
    
- You can create a temp table using SELECT INTO, which can be quicker to write (good for ad-hoc querying) and may allow you to deal with changing datatypes over time, since you don't need to define your temp table structure upfront.
    
- You can pass table variables back from functions, enabling you to encapsulate and reuse logic much easier (eg make a function to split a string into a table of values on some arbitrary delimiter).
    
- Using Table Variables within user-defined functions enables those functions to be used more widely (see CREATE FUNCTION documentation for details). If you're writing a function you should use table variables over temp tables unless there's a compelling need otherwise.
    
- Both table variables and temp tables are stored in tempdb. But table variables (since 2005) default to the collation of the current database versus temp tables which take the default collation of tempdb ([ref](https://learn.microsoft.com/sql/t-sql/language-elements/declare-local-variable-transact-sql)). This means you should be aware of collation issues if using temp tables and your db collation is different to tempdb's, causing problems if you want to compare data in the temp table with data in your database.
    
- Global Temp Tables (##tmp) are another type of temp table available to all sessions and users.
    

Some further reading:

- [Martin Smith's great answer](https://dba.stackexchange.com/a/16386) on dba.stackexchange.com
    
- MSDN FAQ on difference between the two: [https://support.microsoft.com/en-gb/kb/305977](https://support.microsoft.com/en-gb/kb/305977)
    
- MDSN blog article: [https://learn.microsoft.com/archive/blogs/sqlserverstorageengine/tempdb-table-variable-vs-local-temporary-table](https://learn.microsoft.com/archive/blogs/sqlserverstorageengine/tempdb-table-variable-vs-local-temporary-table)
    
- Article: [https://searchsqlserver.techtarget.com/tip/Temporary-tables-in-SQL-Server-vs-table-variables](https://searchsqlserver.techtarget.com/tip/Temporary-tables-in-SQL-Server-vs-table-variables)
    
- Unexpected behaviors and performance implications of temp tables and temp variables: [Paul White on SQLblog.com](https://sql.kiwi/2012/08/temporary-tables-in-stored-procedures.html)