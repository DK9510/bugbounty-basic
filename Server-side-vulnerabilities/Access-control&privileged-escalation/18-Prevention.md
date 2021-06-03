## How to prevent access control Vulnerabilities to Occur 
- Never rely on Obfuscation for access control
- Unless a resource is intended to be publicly accessible, deny access by default
- whenever possible, use single application-wide mechanism for enforcing access controls.
- at the code level, make it mandatory for developers to declare the access that is allowed for each resource, and deny access by default.
- audit and test access controls to ensure they are working as designed.