### Brute-Force 
username & password : rate-limit to prevent and make good password policy.

## Note:
```js
When Server Block user after some attemts we need to spoof ip using 
X-Forwarded-For: (numbers 1 to any) 
```
This help us to spoof out ip and bypass rate-limit Blocking in Brute-Force attack.

## Flawed Brute-Force
 * locking account after some failed attempts
 * Blocking remote user ip address

But if server or system reset the counter of how many request come after successful login 
in this case we can insert out valid credential in some intervals in bruteforce to reset the counter of the system and our ip didn't get blocked by the server.

## NOTE:
if application make requested parameter in JSON format then we pass array of the password list, so we provide multiple password at single time that we can chek all the password and also bypass rate limit and save time.

Request.
```JS
{Username: "carlos",
Password : "passwd";}
```
 Payload for exploit
 ```js
 {Username: "carlos",
 Password: ["passwd1", "passwd2",...."passwdN"];}
 ```
 
 
