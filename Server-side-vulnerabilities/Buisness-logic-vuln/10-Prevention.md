## how to Prevent Business Logic vulnerabilities
- Make sure developers and testers understand the domain that the application serves.
- Avoid making implicit assumptions about user behavior or the behavior of other parts of the application.
- identify the assumption you have made about server-side state and implement the necessary logic to verify these assumption are met.
- Maintain clear design documents and data flows for all transactions and workflows, noting any assumptions that are made at each stage.
- Note any references to other code that uses each component. Think about any side-effects of these dependencies if a malicious party were to manipulate them in an unusual way.
- 