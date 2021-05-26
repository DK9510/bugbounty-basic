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

