#### Broken access control vulnerability exist if a users perform or access some resources that they are not supposed to be able to do.

## Vertical Privilege Escalation
- if user get access to some functionality that they are not supposed to be able to access then it is vertical privilege escalation.
- if non admin user get access to admin functionality.

### Unprotected Functionality
- when an application does not enforce any protection over sensitive functionality.
- example: administrative functions might be linked from an administrative's welcome page btu not from user's welcome page.
- however user might able to access by simply browsing to that admin URL
- `https://vuln-web.com/admin` 	a non admin user might by able to access this and access some functionality 
- this page may be disclosed in	`robots.txt ` or we can brute force it with the common wordlists.

- In some cases, sensitive functionality is not robustly protected but is concealed by giving it a less predictable URL: so called security by obscurity. Merely hiding sensitive functionality does not provide effective access control since users might still discover the obfuscated URL in various ways.
- example: `https://insecure.com/administrator-panel-yb238-mlka`
- but there is may leakage of that URL in the java script that check that user is admin or not to provide him a user interface.

### Parameter-based Access Control Methods
- some applications determine the user's access rights or role at login. and store that information in user-controllable location i.e cookie, or query-string parameter
- it decide the access control based on submitted value
- `https://insecure-web.com/login/home.jsp?admin=true`
- `https://insecure-web.com/login/home.jsp?role=1`
- this is insecure because the user  can modify the value and get access to it.
- some time user role can be modified using update profile and it may be in hidden parameter in JSON format.

### Broken Access control resulting from platform Mis-configuration 
- some application enforce access controls at the platform layer by restricting access to specific URLs and HTTP methods based on user's role
- `DENY: POST, /admin/deleteUser, managers` various things can go wrong in this situation , leading to access control bypass using `custom header`
- like `X-Original-URL: `or `X-Rewrite-URL: `etc
- an alternative attack can arise in relation to the HTTP method used in the request. The Front-end controls above restrict access based on the URL and HTTP Method. 
- if attacker change request mode `GET, POST,POSTX UPDATE` etc to circumvent the access control that is implemented at the platform layer restricted URL;
- we logging in using normal user account and make request to admin access end point after changing request method and we may access to admin functionalities.

## Horizontal privilege escalation
- it arises when a user is able to access to reso1ple : employee have access to his payroll but also have access to other employees also then it is horizontal privilege escalation
- this attack may use similar types of exploit methods to vertical privilege escalation
- `https://vulne-web.com/myaccount?id=9510`
- if we modify the `id ` parameter and able to access other user's account page.
- in some application parameter does not have a predictable value.instead of incrementing number, application might use `GUID(Globally Unique Identifiers ` that leak some where in the application, may be at users are referenced.
- check all the application in that specific user's blog or comment that may `leak id` of that user.
- In some cases, an application does detect when the user is not permitted to access the resource, and returns a redirect to the login page. However, the response containing the redirect might still include some sensitive data belonging to the targeted user, so the attack is still successful.
- `for exploitation: we need to send ?id=wiener to repeater and change id and see resopnse redirecting with leakage of id`

## Horizontal to Vertical Privilege Escalation
- since horizontal priv. esc might allow attacker to compromise admin user since this also an user, so by this this can also be gaining access to admin functionalities and also refer as horizontal to vertical priv. escalation
- If the target user is an application administrator, then the attacker will gain access to an administrative account page. This page might disclose the administrator's password or provide a means of changing it, or might provide direct access to admins data and if they use password reset functionality with the auto filled old password we can note password and get their account.
