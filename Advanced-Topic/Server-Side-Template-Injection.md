# Server Side Template Injection (SSTI)
## what it is 
- SSTI is when when an attacker is able to use `native template sysntax` to inject malicious payload into a template, which is executed server-side.
- template engine design to generate web pages by combining fixed templates with volatile data.
- it is occur when user input is concatenated directly into a template, rather than passed in as data, this allows attacker to inject `arbitrary template directives` in order to manipulate the template engine, often enabling them to take complete control of the server.

## Impact of SSTI vulnerability
- it expose website to variety of attacks depending on the template engine.
- in certain circumtances, these vulnerability poses no real security risk, however most of the time the impact of `SSTI` is catastrophic.
- at severe end of the scale, attacker can achieve `RCE` , taking full control over server.
-  Even in cases where full remote code execution is not possible, an attacker can often still use server-side template injection as the basis for numerous other attacks, potentially gaining read access to sensitive data and arbitrary files on the server.
## How SSTI Arises
- most of time it arise when `user input` directly concatenated into `template` rather than being passed in as data.
- `Static Templates` that simply provide placeHolders into which dynamic content rendered are generally `not vulnerable` to `SSTI`
- `example: ` the email that greets each user by their name, such as `twig template`
```php
$output = $twig->render("Dear {first_name},",array("first_name"=> $user.first_name));
```
this is not vulnerable to `SSTI` because the user's first name is merely passwd in to template as data.
- since template are simply `Strings` web developer sometimes directly concatenate user input into template prior to rendering .
- `Example:` same as previous example but this time user are able to customize parts of the email before it sent, like they might able to choose the name that is used
```php
$output = $twig->render("Dear ".$_GET['name']);
```
in this example value is passed in to template engine using `GET` parameter `name` so this can be exploited by
`https://vuln-web.com/?name={{malicious code }}`

- Vulnerabilities like this are sometimes caused by accident due to poor template design by people unfamiliar with the security implications. Like in the example above, you may see different components, some of which contain user input, concatenated and embedded into a template. In some ways, this is similar to `SQL injection` vulnerabilities occurring in poorly written prepared statements.
- However, sometimes this behavior is actually implemented intentionally. For example, some websites deliberately allow certain privileged users, such as content editors, to edit or submit custom templates by design. This clearly poses a huge security risk if an attacker is able to compromise an account with such privileges.

# Constructing Server Side Template Injection attack
- identifying `SSTI` vulnerabilities and crafting a successful attack typically involves following high-level process
	1. **Detect**
	2. **Identify**
	3. **Exploit:** 
		- **Read**
		- **Explore**
		- **Attack**

## Detect
- Server-side template injection vulnerabilities often go unnoticed not because they are complex but because they are only really apparent to auditors who are explicitly looking for them. If you are able to detect that a vulnerability is present, it can be surprisingly easy to exploit it. This is especially true in `unsandboxed environments.`
- as with any vulnerability, the first step towards exploitation is `FIND it`.
- initial approach is `fuzzing` the template by `special charector ` used commonly in templates.
 ```
 exampls: ${{<[%'"}}%\
 ```
- if an exception is raised , this indicates that injected template syntax  is potentially being interpreted by the server in some way.
- this is one of sign that vulnerability is exist.
- Server-side template injection vulnerabilities occur in two distinct contexts, each of which requires its own detection method. Regardless of the results of your fuzzing attempts, it is important to also try the following context-specific approaches. If fuzzing was inconclusive, a vulnerability may still reveal itself using one of these approaches. Even if fuzzing did suggest a template injection vulnerability, you still need to identify its context in order to exploit it.

### Plaintext Context
- Most template language allow you to freely input content either by using `HTML tags ` directly or by using `template's native syntax`, which will be rendered to HTML on the back-end before the `HTTP` response is sent.
- `Exampls:` the line `render('hello' + username)` would render to `hello DK`.
	- this can some time exploited for `XSS` and is in fact often mistaken for simple `XSS` vulnerability , however by setting mathematically operations as the value of the parameter we can test whether this is also `potential entry point for SSTI` attack.
	- `example:` code `render('hello' +username)` during auditing we might test for `SSTI` by requesting such as
	- `https://vuln-web.com/?username=${7*7}`
	- if resulting output contain `hello 49` this shows that mathematically evaluated in `server side` and this is good POC for `SSTI` vulnerability
	- the `synrax ` is depending on which template engine is used 
	
### Code Context
- in other case, the vulnerability is exposed by `user input` being places within a template expression, as we saw earlier with our email example, this may take form of user controllable variable being placed inside a `parameter` such as
```js
greeting = getQueryParameter('greeting')
engine.render('hello {{"+greeting+"}}', data)
```
on this website resulting URL would look like
`https://vuln-web.com/?greeting=data.username` it would render as hello DK
- this context is missed during assessment, because it doesn't result on obvious `XSS` and is almost indistinguishable from a simple hashdump lookup.
- one method of testing `SSTI` in this Context is to first establish that the parameter doesnt't contain a direct XSS vulnerability by injecting arbitrary `HTML tag`
- `https://vuln-web.com/?greeting=data.username<tag>`
- in the absence of `XSS` this would be either blank entry in the output (just hello with no username), encoded tags or an error message.
- the next step is try and break out of the statement using common templating syntax and attempt to inject arbitrary `HTML` after it
- `https://vuln-web.com/?greetin=data.username}}<tag>`
- if this again resulting `error` or blank output, we have either used `wrong syntax for template engine or language` or if no template-style syntax appears to be valid, then `SSTI` is not possible, Alternatively, if the output is rendered correctly, along with the arbitrary `HTML` this is a key indication that `SSTI` vulnerability is present.

## identify
- once we detected the `SSTI` potential, the next is find or identify the `Template Engine` is in use.
- since there is huge number of templlating language, many of them use very `similar syntax` that is specially chosen not to clash with `HTML charecters`, as result it can be relatively simple to create `probing payload` to test which template engine is being used.
- Simply submitting `invalid syntax` is often enough, because the `resulting error message` will tell you exactly what the `template engine` is and some times which `version` is being used also.
- `exampls:` `<%=foobar%>` triggers following response from `ruby-based ERB engine`
```rb
(erb):1:in`<main>`: undefined local variable or method `foobar` for main:Object (NameError)
from /usr/lib/ruby/2.5.0/erb.rb:876:in `eval'
from /usr/lib/ruby/2.5.0/erb.rb:876:in `result'
from -e:4:in `<main>'
```

- otherwise we need to manually test different `language-specific` payloads and study how they are interpreted by the template engine.
- using process of elimination based on which syntax to be valid or invalid, we narrow down the options quicker.
- A common way of doing this is to inject arbitrary mathematical operations using Syntax from different template engines.
- we use decision tree similar to the following

![[SSTI_dicision_tree.png]]
![](https://github.com/DK9510/Img/blob/main/SSTI_dicision_tree.png)
- also be aware that same payload can sometimes return a `successful` response in more than one template language.
- example: `{{7*7}}` returns `49` in twig and `7777777` in jinja2, therefor it is important to not jump to conclusion based on a single successfull response.

# Exploiting SSTI
## Read 
- unless we know the template engine inside out, reading its documentation is usually the first place to start.
- reading documentation may some times save our time and effort in exploitation.
## Learning the Basic template Syntax
- learning the basic syntax is obviously important, along with key functions and handling of variables.
- Even something as simple as learning how to embed native code blocks in the template can sometimes quickly lead to an exploit.
- example: `once we know that python-based Mako template engine is being used, achieving RCE could be simple as:`
```python
<% 
import os
x=os.popen('id').read()
%>
${x}
```
in an Unsandboxed environment, achieving `RCE` and using It to `read, edit or delet` arbitrary files is similarly as simple in many common template Engines.
- after finding which `template engine` is being used than find syntax for exploiting 
- ruby EBR have one method called `system('system command')` which used to gain `RCE` in the server.

#### while exploiting Context based SSTI find where the user.name or context data is rendered in the application in order to exploit it.
exampls: in blog user have `user.name `context field where he chose what name it want to display and that based on that the comment posted by user is got the name to display there.

## Read about security Implications
- in addition to provide the fundamental of how to create and use templates, the documentation may also provide some sort of `Security section`.
- in this section they provide name of some `dangerous function ` that avoid during implementing template in website or application.
- even there is no dedicated `security section`, if a particular built-in object or function can pose a security risk, there is almost always a `warning of some kind` in the documentation.
- example: `in ERB Ruby template engine , documentation revel that we list all directories and read arbitrary files as follows: `
```erb
<%= Dir.entries('/') %>
<%= File.open('/example/arbitrary.file').read %>
```

- some time we need to dig deep in order to find and exploit, simply understand documentation of template engine and also read `FAQ` if it have and try to find way to `insert, create, import, define moduls and object ` that will help us to exploitation easily. 
- `NOTE` look closely to errors because they give us the information about which template engine is running etc.



## Look For Known Exploits
- some time look for template engine and try to search for known exploits that have been exploited by some hunter while hunting and try that exploit and if necessary modify the exploit according to template engine.

## Explore
- at this point we might have already stumbled across workable exploit using the `documentation`,  the next step is to explore the environment and try to discover all the objects to which you have access.
- Many template engine expose `self` or `environment` object of some kind, which is act like a namesspace containing all `objects, methods and attributes` that are supported by the template engine, if such an object exist, we potentially use it to generate a `list of objects that are in scope` 
- example: java-based templating language, sometimes we can list all variables in the environment using following injection
```java
${T(java.lang.System).getenv()}
```

`NOTE`: in **BURP suite professional we have built in payload list for various vulnerability in intruder section like template injection checker and server side variable list etc**

## Developer Supplied Objects
- it is important to note that application contains both `built in objects ` provided by the template and `Custim, site-specific objects` that have been supplied by the `web developer`. 
- we pay particular attention to these non-standard objects because they are especially likely to contain `sensitive information` or `exploitable methods`.
- As these objects can vary between different templates with in the same website, be aware that you might need to study an object's behavior in the context of each distinct template before you find a way to exploit it.
- since `SSTI` can potentially lead to `RCE` and full takeover of the server, in practical this is not always possible to achieve `RCE`, that doesn't mean there is no potential for other `exploits`, we can still leverage `SSTI` vulnerability for other `High-saverity exploits` such as `Directory-Traversal or gain access to sensitive data.`

[LAb portswigger](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-information-disclosure-via-user-supplied-objects) information disclosure via user-supplied objects in SSTI
- from error we got this is `Django` template used, see the documentation and got `{% debug %}` , gives us list of objects and methods, use documentation and find sensitive info using that objects and methods.

# Creating Custom Attack
- some times we need to construct our own attack and exploit in order to gain access to the system and this often happened when `template engine is run in sandbox environment`, which make exploitation difficult or even impossible.
- after identifying attack surface, if there is no obvious way to exploit the vulnerability, we proceed with traditional techniques by `reviewing each function for exploitable behaviour,` by working methodically through this process.
- sometimes we need to construct complex attack that is even able to exploit more secure targets.

## Creating Custom Exploit using Object Chain.
1. identify the `objects` and `methods` to which we have access, some objects may immediately jump out interesting. By combining knowledge we have and using documentation we sort list objects that we need to investigate.
2. while studying documentation, attention to which methods these objects grant access to, as well as which objects they return.
3. By go deep in documentation , we discover `combination of objects and methods` that we chain together. Chaining together right objects and methods may allow us to access `dangerous functionality` and `sensitive data` that initially appear to be out of reach.
`example:` java based template engine `velocity`, we access to `ClassTool` object called `$class`. studying the documentation revels that you can chain the `$class.inspect()` method and `$class.type` property to obtain references to `arbitrary object`.
in past, this has been executed to shell commands on the target system
```
$class.inspect("java.lang.Runtime").type.getRuntime().exec("system commands")
```
like java have method to find class of object `object.getClass()` investigate and read documentation of that class and find interesting methods that can we use to chain and exploit it .

## Creating Custom Exploit using Developer Supplied Objects
- some template engines run in a secure, locked-down environment by default in order to mitigate the associated risks as much as possible. 
- This makes it difficult to exploit such as RCE, developer created objects that are Exposed to the template can offer a further, less battle-hardened attack surface.
- However documentation is usually provided for template built-ins, the `site-specific` objects are almost certainly not documented at all.
- so for working out how to exploit them will require us to investigate the website/application's behavior manually to identify the attack surface and construct owr own custom exploit accordingly.
- `Nots:` try to combine vulnerability with other functionality
- `example:` in [portswigger lab](https://portswigger.net/web-security/server-side-template-injection/exploiting/lab-server-side-template-injection-with-a-custom-exploit)
- if we upload another file than image we got error and this revel `user.setAvatar()`
 method from which we get `LFI` and after analyzing source code from `LFI` we know the method `gdprDelete()` which delete the file we `set as avatar` and calling that method delete the file.


# [FOR additional resource in SSTI](https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection)

# Prevention
- best way to prevent `SSTI` is to not allow user to modify or submit new templates, 
- always use logic less templates, such as `mustache`
- always execute user's code in `sandbox` environment, where potential dangerous modules and function are removes.
