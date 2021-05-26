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
