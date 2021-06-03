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