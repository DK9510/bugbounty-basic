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

