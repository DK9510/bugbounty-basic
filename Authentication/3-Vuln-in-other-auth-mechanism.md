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
	


