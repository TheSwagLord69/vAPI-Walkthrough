> Nothing has been logged or monitored , You caught us :( !

# Tools Used:
```
curl 8.8.0 (x86_64-pc-linux-gnu)
```

# Get Flag
## GET /vapi/api10/user/flag

Get the flag.
```bash
curl -i http://localhost/vapi/api10/user/flag
```
```http
HTTP/1.1 200 OK
Host: localhost
Date: Sun, 30 Jun 2024 19:16:57 GMT
Connection: close
X-Powered-By: PHP/7.4.33
Cache-Control: no-cache, private
Date: Sun, 30 Jun 2024 19:16:57 GMT
Content-Type: application/json

{"message":"Hey! I didn't log and monitor all the requests you've been sending. That's on me...","flag":"flag{api10_5db611f7c1ffd747971f}"}
```

API10 Flag
```
flag{api10_5db611f7c1ffd747971f}
```

References:
https://owasp.org/API-Security/editions/2019/en/0xaa-insufficient-logging-monitoring/