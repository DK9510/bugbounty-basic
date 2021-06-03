* In any OAuth flow, the user must approve the requested access based on the scope defined in the authorization request. The resulting token allows the client application to access only the scope that was approved by the user. But in some cases, it may be possible for an attacker to "upgrade" an access token (either stolen or obtained using a malicious client application) with extra permissions due to flawed validation by the OAuth service. The process for doing this depends on the grant type.

### Scope Upgrade: 
* their is chance to manipulate the scope
* also unverified user registration that doesn't verify the user who register so attacker can register with the same detail that victim have and may able to login in as victim.
