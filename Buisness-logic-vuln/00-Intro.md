## [what are Business Logic vulnerability](https://portswigger.net/web-security/logic-flaws)
* Business logic vulnerabilities are flows that arise in the design and implementation of an application that allow an attacker to elicit Unintended Behavior. This potentially enables attackers to manipulate legitimate functionality to achieve a malicious goal.
* Logic flaws are often invisible to people who aren't explicitly looking for them as they typically won't be exposed by normal use of the application. However, an attacker may be able to exploit behavioral quirks by interacting with the application in ways that developers never intended.
* Flaws in the logic can allow attackers to circumvent these rules. For example, they might be able to complete a transaction without going through the intended purchase workflow.
* the business logic vulnerability is unique and diverse depending on the target application, so it needed manual testing because it is very difficult to detect using automated scanner.

## how it arise 
* Business logic vulnerabilities often arise because the design and development teams make flawed assumptions about how users will interact with the application.
* Logic flaws are particularly common in overly complicated systems that even the development team themselves do not fully understand.
* Developers working on large code bases may not have an intimate understanding of how all areas of the application work.

## Impact OF logic vulnerabilities
* the impact of business logic vulnerabilities is depend on the application and which logic or area of the application are broken.
* it depend on the functionalities.
* eg. the flow in authentication then it is high severity due to it risks overall security.
