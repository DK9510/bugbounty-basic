#**Authentication** It is the Most critical Vulnerability due to access of confidential data

![AUTHENTICATION](https://github.com/DK9510/Img/blob/main/Authentication.png)

## Intro
Process to identify the eligibility of the user before access confidential data.
## Authentication: 
```
Process that verify that a user is who they claim it to be.
```
## Authorization:
```
Process of verifying user is allowed to do something.
```

### how it Arise
* because they fail to protect against Brute-Force.
* logic flows or poor coding bypassed by attacker known as "Broken Authentication".

## Vulnerability in Authentication Mechanism
* Vulnerability in Password Based Login
* Vulnerability in Multi-Factor authentication
* Vulnerability in Other Authentication Mechanism

# password based login
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
```js
{Username: "DK9510",
Password : "Th3Most$#cuRePasswd";}
```
 Payload for exploit
 ```js
 {Username: "DK9510",
 Password: ["passwd1", "passwd2",...."passwdN"];}
 ```


# Multi-factor authentication

## 2FA Bypass
if user promoted for password than and then prompted to verification code through different page.
The user is effectively in "logged in " state before they have entered the verification code, we can skip "loged in only " pages, and sys doesn't check if we completed the 2 nd step or not.

### exploit 
```bash
simply login in 1st page and then change the URL to valid URL that
gives after we enter the verification for one valid User, and we 
successfully bypassed 2FA.
```

## Flowed 2F verification Logic
* flow is after completing initial login step, the user doesn't verify that the same user completed 2 nd stage.
* we login using our creds and when we need to vary with verification code we change the user with other user id, and we can manipulate that.
* some time we need to brute force the verification code but we didn't need any password for victim user.


## 2FA Broken Logic
* Try to login with valid creds.
* if the request contain field to verify user name in the cookie.
* resend that request with victim user name so we generate verification code for victim in the server.
* login with our creds and when we enter OTP then change the verifying cookie to victim and brute-force OTP and we got their account without their password.

```bash
some application privent brute forcing or block or reset the page
to enter user name and password then we can bypass is with help of
" MACROS". 
```

# Other authentication mechanism
* Vulnerability in other mechanism or Functionality like Password Reset or Forgot Password functionality.

### keep user loged in 
* "Remember me " or "keep  me loged in " with the help of this cookie we can bypass entire process.
*  this functionality is vulnerable if the cookie of for session keeping is generated or part of the cookie contain password of the user. then attacker can reverse decryption and brute-force cookie and also got the victims password.

### stored-xss + stay-logged in-cookie

* application have stored xss we can steal other users cookie and access their account.

### password reset broken logic 
* when server send password reset token and  we reset password that time server DOESN'T validate token and user name so we manipulate and change the password of victim.
[example](https://portswigger.net/web-security/authentication/other-mechanisms/lab-password-reset-broken-logic)
### changing user Password 
* changing your password involves entering your current password and then new password 2 time. 
* this page check the user name and password before changing so it may also have vulnerability.
* it dangerous if attacker DIDN'T need to login for change password 
* it may happens when attacker send password reset request and it contains user name in the hidden field so it can be manipulated and attacker controls the victim account.
* analyze the password change request and try to manipulate it.
	
