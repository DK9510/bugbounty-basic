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
