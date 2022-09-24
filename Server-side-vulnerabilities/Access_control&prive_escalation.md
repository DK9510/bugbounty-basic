# Access control

![](https://github.com/DK9510/Img/blob/main/Access-Control.png)
![[Access-Control.png]]
## what is Access control
- access control(or authorization) is the application of constraints on who can perform what actions on the application or what actions can be performs by who and give access to the resources that they requested, in the context of the application.
- it is dependent on authentication and session management.
- ***Authentication:*** Identifies the users and confirms that they are who they say they are.
- ***Session Management:*** identifies which subsequent HTTP requests are being made by that same user.
- ***Access Control:*** Determines weather the user is allowed to carry out the action that they are attempting to perform on the application.

- `Broken Access Control` are common and critical security vulnerability.

## Access controls can be divided into following 

- Vertical Access controls
- Horizontal Access controls
- Context-Dependent access controls

# Access Control Models

## what are Access Control Security Models
- it is definition of a set of access control rules that is independent of technology or implementation platforms.
- it can be implemented with in Operating System, Networks, database management systems, back office, application, web server software etc.

various access control models
- Programmatic access control
- Discretionary access control(DAC)
- Mandatory access control(MAC)
- Role-based access control(RBAC)

## Programmatic Access Control
- a matrix of user privileges is stored in a database and access controls are applied programmatically with the referance to the matrix.
- this can include Roles or Groups or individual users

## Discretionary access Control (DAC)
- in this the access to resources or functions is constrained based upon users or named groups of users.
- owner has ability to grant and remove access permission of other users.
- this model is very complex to design and manage.

## Mandatory access control(MAC)
- it is centrally controlled system of access control in which access to some object by a subject is constrained.
- unlike DAC the users and owner of the resources have no capabilities to modify the access rights for anyone to their resources.

## Role-based access Control(RBAC)
- in this roles are defined to which access privilege are assigned.
- users then assigned to single or multiple roles.
- it provides enhanced management then other models.
- It is more effective when there are sufficient roles to properly invoke access control but not so many, because it make it excessively complex and very hard to manage.

# Access Control Types

## Vertical Access Controls
- it is mechanisms that restrict access to sensitive functionality or Resources that is not available to other types of users.
- diff. types of users have diff application functions `example : admin have rights to delet others acc. but others have no access to delete any other's acc.`
- Vertical access controls can be more fine-grained implementations of security models designed to enforce business policies such as separation of duties and least privilege.

## Horizontal access controls
- it restricts access to resources to the users who are specifically allowed to access those resources.
- eg. in banking application the perticular user only see his account's balance and transaction not other user's.

## Context-dependent access controls
- it restricts the functionality and resources based upon the state of the application or user's interaction with it .
- it prevent the users from performing wrong order.
- example: a retail application might prevent users from modifying the content of their shopping cart after they have made payment.

# Broken Access Control
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

# IDOR
## Insecure Direct Object Reference(IDOR)
- IDOR are a subcategory of access control vulnerabilities. 
- it arise when a user supplied input to access objects directly and an attacker can modify the input to obtain unauthorized access.

## what is IDOR
- Insecure Direct Object Reference are a type of access control vulnerability that arise when an application uses user-supplied input to access objects directly.
- it is most commonly associated with the horizontal privileged escalation, but can also arise in relation to vertical privilege escalation.
-`examples:`
### IDOR Vulnerability with direct reference to database objects.
- consider app that retrieve information from back-end DB using `https://insecure-wen.com/myaccount?customer_no=9510` etc
- here customer number is used directly as record index in queries that performed in back-end so if attacker manipulate number or modify that index number in input he got access to other user's records.

### IDOR vulnerability with Direct reference to static Files
- it arise when sensitive resource are located in static files on the server-side filesystem.
- example:
- `https://insecure.web.com/static/9510.txt` we user manipulate number of that txt file then he retrieve transcript of another user.

# examples of Access Control vulnerability 
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

# prevent access control Vulnerabilities to Occur 
- Never rely on Obfuscation for access control
- Unless a resource is intended to be publicly accessible, deny access by default
- whenever possible, use single application-wide mechanism for enforcing access controls.
- at the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.
- audit and test access controls to ensure they are working as designed.
