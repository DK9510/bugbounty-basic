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