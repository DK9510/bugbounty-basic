## access control in multi-step process 
- many application implements important functions over a series of steps.
- this happened when user needs to review and confirm what he entered or when users need to give variety of inputs to the application.
- `example: `
- administrative function to update user details might involve following steps
	1. Load form containing details for a specific user.
	2. submit change or submit the request to change
	3. review the change and then confirm.
- here if access control are properly applied to 1st and 2nd stage but not in 3rd stage, here attacker gain unauthorized access to the function by skipping the first two steps.
- the application assume that user reach the 3rd step after completing previous 2 steps, and gives permission to user to access these functionalities.

## Referer-Based access control
- some application implement access control in `referer` header submitted in the HTTP request.
- this `referer` header is generally added by browser to indicate the page which a request was initiated.
- `example: `
- application strongly enforce access control in `/admin` but for sub-pages such as `/admin/deleteUser` only inspect the `referer`	header and see if it contained `/admin` then grant access to request.
- in this `referer` header is fully controlled by attacker and forge direct request to `sensitive sub-pages` supplying the required `referer` header and gain access to unauthorized resource.
- 