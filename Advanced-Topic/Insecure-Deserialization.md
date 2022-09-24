# Insecure Deserialization
![[deserialization.png]]
![](https://github.com/DK9510/Img/blob/main/deserialization.png)
## What is Serialization
- serialization is process of converting complex data structure, such as objects and their field, n to a `flatter` format that can be sent and received as sequential `stream of bytes`. 
- serializing data
	- write a complex data to inter-process memory , a file or a database
	- send complex data, for example over a `network`, between different components of an application, or in an `API call`
- when serializing an object its state is also persisted, in other words , the object's attributes are preserved along with their assigned values

## Serialization vs Deserialization
**Deserialization:** 
-  it is the process of restoring this byte stream to a fully functional replica of the `original object`, in the exact state as when it was serialized.
-  The website's logic can then interact with this `deserialized object` just like it would with any other object

![[insecure_deserialization.png]]	
![](https://github.com/DK9510/Img/blob/main/Pasted%20image%2020210614180921.png)	


- many programming language often native support for serialization
- Exactly how objects are `serialized ` depends on the language.
- some language serialize object into a binary formats, where as others use different string formats, with varying degree of human readability.
- Note that `original attributes` are stored in the serialized data stream, including private fields.
- some language serialization referred different name, example `marshalling in (RUBY)` and `Pickling in (PYTHON)` these terms are synonymous with `Serialization `in the context

## Insecure Deserialization 
- `insecure deserialization` is when user-controllable data is deserialized by a `web site or application`.
- this potentially enables an attacker to manipulate serialized objects in order to pass harmful data into the application code.
- it is even possible to replace a serialized object with an entirely different class.
- Alarmingly, Object of any class that is available to the application will be deserialized and instantiated, regardless of with class was expected, for this reason `insecure deserializatoin` is also known as `object injection vulnerability`.
- An object of an unexpected class might cause an `exception`. by this time, however the damage may already be done.
- Many `deserialization attacks` are completed `before` deserialization is finished. This means that the deserialization process itself can `initiated attack`, even if application's own functionality does not directly interact with the malicious object.
- This is the Reason, website or application whose logic is based on Strongly Types language can also be vulnerable to these techniques

## How Insecure Deserialization Vulnerability arise 
- it is arise due to general lack of understanding of how dangerous `deserializing` user-controllable data can be. ideally user input should never be deserialized at all.
- some time owner thinks they are safe because they implement some form of additional check on the deserialized data. this approach is often ineffective because it is virtually `impossible to implement validation or sanitization` to account for every eventuality.
- these checks also fundamentally flowed as they rely on checking the data after it has been deserialized, which in many case too late to prevent the attack.
- vulnerability may also arise because `deserializes objects` are often trustworthy. Especially when using languages with a binary serialization format, developers thought that users can not `read or manipulate ` data effectively.
- However it may require more effort, it is as possible for an attacker to exploit binary serialized objects as it is to exploit string-based formats
- `deserialization-based attack` are also made possible due to the number of dependencies that exist in modern web application. 
- typically application might implement many different libraries, which each have their own dependencies as well.  these create a massive pool of classed and methods that is difficult to manage securely. As an attacker can create instances of any of these classes.
- it is hard to predict which methods can be invoked on the malicious data. This is especially true if an attacker is able to chain together a long series of unexpected method invocations, passing data into a sink that is completely unrelated to the initial source. It is, therefore, almost impossible to anticipate the flow of malicious data and plug every potential hole.
- in short it state that 99% not possible to securely `deserialize untrusted input`

## Impact of Insecure deserialization vulnerabilities
- The Impact of `insecure deserialization` can be vary severe because it provides an entry point to a massively increased attack surface.
- it allows an attacker to reuse existing application code in harmful ways, resulting in numerous other vulnerabilities, often `RCE(Remote Code Execution)`
- even in case `RCE` is not possible it can lead to `Privilege escalation, Arbitrary File access, DOS attacks`

# Exploiting Insecure Deserialization Vulnerabilities

## How to Identify Insecure Deserialization 
- during auditing, look at all data being passed into the website and try to identify anything that looks like `serializaed` data.
- Serialized data can be identifies relatively easily ,by knowing the `format that different languages use.`

## PHP Serialization Format
- PHP uses a mostly human-readable string format, with letters representing the data type and numbers representing the length of each entry.  
`Example: User object with attributes`
```php
$user->name = "DK9510";
$user->isLoggedIn = true;
```
when serialized, this looks like this
`0:4:"User:2:{s:4:"name":s:6:"DK9510";s:10:"isLoggedIn":b:1;}`
this can be interpreted as 
- 0:4:"User"  an object with the 4 character class name "User"
- 2  the object has 2 Attributes
- s:4:"name"  The key of the first attribute is the 4 - char string "name"
- s:6:"DK9510"  the value of the first attribute is the 6 - char string "DK9510"
- s:10:"isLoggedIn"   the key of the second attribute is 10- char string "isLoggedIn"
- b:1  the value of the second attribute is the boolean value true

**Native method for serialization and deserialization**
`serialize()` and `unserialize()`
if we have source code access, we search for `unserialize()` anywhere

## Java Serialization Format
- some language such as java, use `binary serialization formats`. this is more difficult to read, but still we identify serialized data, if know how to recognize few `tell-tale signs`
- `example:` serialized java objects always begin with the `same bytes` which is encoded as `ac ed` in hex and `r00` in base64
- the interface `java.io.Serializable` can be used to `serialize and deserialize` .
- The `readObject()` method which is used to read and deserialize data from `InputStream`

## Manipulation serialized Objects
- Exploiting some `deserialization vulnerability` can be as easy as changing an attribute in a serialized object.
- as the object state is persisted, you can study the serialized data to `identify and edit` interesting attribute values.
- we pass the malicious object into the website via its deserialization process. this is initial steps for `deserialization exploit`
- there is two approaches for `manipulating serialized objects`
	- edit the object directly in its byte stream from 
	- write a short script in the corresponding language to create and serialize the new object your self. this will easy when approaching `binary serialization formats`

## Modifying Object attributes
- when tempering with the data, as long as the attacker preserves a `valid serialized object`, the deserialization process will create a server-side object with the modified attribute values
- `example:` application that use `serialized User object` to store data about a user's session in a cookie, if an attacker spotted this serialized object in `HTTP request`, they might decode it to find the following `Byte stream`
`O:4:"User":2:{s:8:"username";s:6:"DK9510";s:7:"isAdmin";b:0;}`
- the `isAdmin` attribute is an obvious point of interest.  attacker simply change the boolean value of the attribute to `1 (true)` re-encode the object and overwrite their current cookie , in isolation this had no effect but when application use cookie to check whether user is admin or not 
```php
$user=unserialize($_COOKIE);
if ($user->isAdmin == true){
//allow access to admin interface}
```

but this simply scenario is not common in the wild , but editing attribute value in this way demonstrates the first step towards accessing the massive amount of attack-surface exposed by insecure deserialization

## Modifying Data Types
- we seen how you can modify attribute values in serialized object, but it's also possible to supply unexpected data types.
- usually, this also works for any alphanumeric string that starts with `number` , in this case `PHP` will effectively convert the entire string to an integer value based on the initial number. the rest of the string is ignored completely.
- `5 == "5 of something " ` is in practice treated as `5 == 5`.
- this becomes stranger when comparing a string the integer "0"
 ` 0 == "Example string" ` //true
 because there is no number, that is  0 numerals in the string, PHP treats this entire string as the integer 0
  - `example:` consider a case where this loose comparison operator is used in conjunction with user-controllable data from a deserialized object, and this result in `potential logic flows`

```PHP
$login = unserialize($_COOKIE)
if ($login['password']== $password)
{//login sucessfully
}
```
- attacker modified the password attribute so that it contained the integer `0` instead of the expected string
- as long as the stored password does not start with a number, the condition would always return true, enabling an authentication bypass.
- this is only possible if application uses deserialization, because deserialization `preserves data type ` , but if it directly fetch the password then `0` converted in to string and condition evaluate to `False`
- `NOTE:`
Be aware that when modifying data types in any serialized object format, it is important to remember to update any type labels and length indicators in the serialized data too. Otherwise, the serialized object will be corrupted and will not be deserialized.
`Exploit:` if some application use serialized cookie which compare `auth_token ` in string format we exploit it using `change the data type of auth_token value and change it to i ' i.e integer ' and in value put 0 ` this will always evaluate as true
Original cookie:
```php
O:4:"User":2:{s:8:"username";s:6:"wiener";s:12:"access_token";s:32:"abmx0nljzdt17g8yd5c4j31jab3zj00v";}
```
after modifying data type in serialized object exploit:
```php
O:4:"User":2:{s:8:"username";s:13:"administrator";s:12:"access_token";i:0;}
```

## Using Application Functionality
- as Well simply check Attribute values, a website's functionality might also perform dangerous operations on a data from operations on data from a deserialized object. 
- in this case we use insecure deserialization to pass in unexpected data and leverage the related functionality to do damage.
- `example:` as a part of website's `Deleter User` functionality, the user's profile picture is deleted by accessing file path in the `$user->image_location` attribute , if this `$user` was created from serialized object, an attacker could exploit this by passing in a modifies object with the `image_location` set to an arbitraty file path.
- Deleting their own user account would then delete this arbitraty `file` as well.

## Magic Method
- magic  methods are a special subset of methods that you do not have to explicitly invoke. instead they are invoked automatically whenever a particular event of scenario occur.
- Magic methods are common feature of object-oriented programming in various languages.
- they are sometime indicated by prefixing or surrounding the method name with double-underscore(\_\_)
- developers cad add magic methods to a class in order to predetermine what code should be executed when the corresponding event or scenario occurs.
- `exampls:`  in PHP 
  `__construct()`, which is invoked when an object of the class is instaniated, similar to python's `__init__`
- typically, constructor magic methods like this contain code to initialize the attribute if the instance , how ever magic methods can be customized by developers to execute any code they want
- `Magic methods` are widely used and do not represent a vulnerability on their own, but they can become dangerous when the code that they execute handles attacker-controllable data, `example:` a `deserialized object`, this can be exploited by an attacker to automatically invoke methods on the `deserialized data` when a corresponding conditions are met.
- Most importantly in this context, some languages have magic methods that are invoked `automatically during ` the `deserialization process`.
- `example:` php's `unserialize()` method looks for and invokes an object's `__wakeup()` magic method.
- java `deserialization` the same applies to the `readObject()` method, which essentially act's like constructor for `reinitializing` a serialized object. 
- `ObjectinputStream.readObject()` method is used to read data from the initial byte stream, however serializable classes can also declare their own `readObject()` methods as follow
```java
private void readObject(ObjectInputStream in) throws IOExeception, ClassNotFoundExeception{...};
```
- this allows the class to control the deserialization of its own fields more closely. Crucially , a `readObject()` method declared in exactly this way acts as a `magic method` that invoked during deserialization.
- You should pay close attention to any classes that contain these types of magic methods. They allow you to pass data from a serialized object into the website's code before the object is fully `deserialized`. This is the starting point for creating more advanced exploits.

## Injecting Arbitrary Objects
- as we seen, it is occasionally possible to exploit `insecure deserialization` by simply editing the object supplied by the website, how ever injecting arbitrary object types can open up many more possibilities
- in `object-orieantedprogramming` , the methods available to an object are determined by its class. therefor, if an attacker can manipulate which class of object is passed in as `serialized data`, they can influence what code in executed after and even  during `deserialization`
- Deserialization methods do not typically check what they are deserializing. this means that you can pass in objects of any serializable class that is available to the website, and object will be deserialize
- This allows attacker to create instance classes. The fact that this Object is not of the expecteed class does not matter. 
- this unexpected object type might be cause `exception` in the application logic, but the `malicios object` already be installed
- if attacker has access to `source code`, they study all the available classes in detail, to construct a simple exploit. they look for a class containing `deserialization megic method` , then check whether any of them perform dangerous operation on controllable data, then attacker can pass in a `serialized object` of this class to use its magic methods for an exploit

### LAB PORTSWIGGER Arbitrary Object Injection
**NOTE:** some time we have access to source code by just append `~` in filename to retrieve an editor-generated backup file `append ~ at end of the file.`
1.  Log in to your own account and notice the session cookie contains a serialized PHP object.
2.  From the site map, notice that the website references the file `/libs/CustomTemplate.php`. Right-click on the file and select "Send to Repeater".
3.  In Burp Repeater, notice that you can read the source code by appending a tilde (`~`) to the filename in the request line.
4.  In the source code, notice the `CustomTemplate` class contains the `__destruct()` magic method. This will invoke the `unlink()` method on the `lock_file_path` attribute, which will delete the file on this path.
5.  In Burp Decoder, use the correct syntax for serialized PHP data to create a `CustomTemplate` object with the `lock_file_path` attribute set to `/home/carlos/morale.txt`. Make sure to use the correct data type labels and length indicators. The final object should look like this:  
    `O:14:"CustomTemplate":1:{s:14:"lock_file_path";s:23:"/home/carlos/morale.txt";}`
6.  Base64 and URL-encode this object and save it to your clipboard.
7.  Send a request containing the session cookie to Burp Repeater.
8.  In Burp Repeater, replace the session cookie with the modified one in your clipboard.
9.  Send the request. The `__destruct()` magic method is automatically invoked and will delete Carlos's file.
- Classes containing these deserialization magic methods can also be used to initiate more complex attacks involving a long series of method invocations, known as a `gadget chain`.

## Gadget Chains
- A `gadget` is snippet of code that exist in the application that can help an attacker to achieve a particular goal
- an individual gadget may not directly do anything harmful with user input.
- However the attacker's goal might simply to be invoke a method that will pass their input to another `gadget` by chaining multiple gadgets together in this way, an attacker can pass input into a `dangerous sink gadget`.
-  It is important to understand that, unlike some other types of exploit, a gadget chain is not a payload of chained methods constructed by the attacker. All of the code already exists on the website. The only thing the attacker controls is the data that is passed into the gadget chain. This is typically done using a magic method that is invoked during deserialization, sometimes known as a `kick-off gadget`.
-   many insecure deserialization vulnerabilities will only be exploitable through the use of gadget chains. This can sometimes be a simple one or two-step chain, but constructing high-severity attacks will likely require a more elaborate sequence of object `instantiations and method invocations`. Therefore, being able to construct `gadget chains` is one of the key aspects of successfully insecure deserialization.
## Working with Pre-built gadget chains
- manually identifying gadget chains is tough process and with out source code it is `impossible` but we use pre-built gadget chains that are discovered and successfully exploited , 
- we use this kind of tools to identify and exploit insecure deserialization vulnerabilities wit little effort, this approach is made possible due to widespread use of `Libraries` that contain exploitable gadget chains.
- `exampls:` gadget chains in `java's apache commons library` can be exploited in any website that used that library.
### Ysoserial 
- it is tools for java deserialization. 
-  This lets you choose one of the provided gadget chains for a library that you think the target application is using, then pass in a command that you want to execute. 
-  It then creates an appropriate serialized object based on the selected chain. 
-  This still involves a certain amount of trial and error, but it is considerably less labor-intensive than constructing your own gadget chains manually.
`example LAB ` [lab_java_deserialization_gadget chain](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-java-deserialization-with-apache-commons)
- use `Ysoserial` tool for finding pre-exploited gadget chain and encode it to `base64 ` then `URL encode` entire payload
`NOTE:` all the `gadget chians` in `ysoserial` enable us to run `arbitrary code` , instead they may use full for another purpose.
- The `URLDNS` chain triggers a DNS lookup for a supplied URL. Most importantly, it does not rely on the target application using a specific vulnerable library and works in any known Java version. This makes it the most universal gadget chain for detection purposes. If you spot a serialized object in the traffic, you can try using this gadget chain to generate an object that triggers a DNS interaction with the Burp Collaborator server. If it does, you can be sure that deserialization occurred on your target.
- `JRMPClient` is another universal chain that you can use for initial detection. It causes the server to try establishing a TCP connection to the supplied IP address. Note that you need to provide a raw IP address rather than a hostname. This chain may be useful in environments where all outbound traffic is firewalled, including DNS lookups. You can try generating payloads with two different IP addresses: a local one and a firewalled, external one. If the application responds immediately for a payload with a local address, but hangs for a payload with an external address, causing a delay in the response, this indicates that the gadget chain worked because the server tried to connect to the firewalled address. In this case, the subtle time difference in responses can help you to detect whether deserialization occurs on the server, even in blind cases.

## PHP Generic Gadget chains
for php based site we use `PHP Generic Gadget Chains(PHPGGC)`
[lab portswigger ](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-exploiting-php-deserialization-with-a-pre-built-gadget-chain)
- exploitation: identify the target framework, and use third party tool to generate malicious serialized object, `generate valid signed cookie` containing malicious `object` 
- if we modify the token we got `500` internal server error at some point and this gives us `version number and which framework is used by application`
```
Internal Server Error: Symfony Version: 4.3.6</p>
                    <p class=is-warning>PHP Fatal error:  Uncaught Exception: Signature does not match session in /var/www/index.php:7
Stack trace:
#0 {main}
  thrown in /var/www/index.php on line 7</p>
 <!-- <a href=/cgi-bin/phpinfo.php>Debug</a> -->
```
here developer comment disclose `debug` page 
- from `debug` page we got `SECRET_KEY` in `Enviornment`
- `SECRET_KEY: key(eg: oy93ozy4qspj3ywruamc3j9bww7xl3kn)` this we use to sign our object in serialization
- we use `phpggc` to create serialize object for gadget chain
create signed cookie so cant get error in server side
```php
<?php  
$object = "OBJECT-GENERATED-BY-PHPGGC";  
$secretKey = "LEAKED-SECRET-KEY-FROM-PHPINFO.PHP";  
$cookie = urlencode('{"token":"' . $object . '","sig_hmac_sha1":"' . hash_hmac('sha1', $object, $secretKey) . '"}');  
echo $cookie;
```
final payload looks like
```
%7B%22token%22%3A%22Tzo0NzoiU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxUYWdBd2FyZUFkYXB0ZXIiOjI6e3M6NTc6IgBTeW  
1mb255XENvbXBvbmVudFxDYWNoZVxBZGFwdGVyXFRhZ0F3YXJlQWRhcHRlcgBkZWZlcnJlZCI7YToxOntpOjA7TzozMzoiU3ltZm9ueVxDb21w  
b25lbnRcQ2FjaGVcQ2FjaGVJdGVtIjoyOntzOjExOiIAKgBwb29sSGFzaCI7aToxO3M6MTI6IgAqAGlubmVySXRlbSI7czoyNjoicm0gL2hvbW  
UvY2FybG9zL21vcmFsZS50eHQiO319czo1MzoiAFN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcVGFnQXdhcmVBZGFwdGVyAHBvb2wi  
O086NDQ6IlN5bWZvbnlcQ29tcG9uZW50XENhY2hlXEFkYXB0ZXJcUHJveHlBZGFwdGVyIjoyOntzOjU0OiIAU3ltZm9ueVxDb21wb25lbnRcQ2  
FjaGVcQWRhcHRlclxQcm94eUFkYXB0ZXIAcG9vbEhhc2giO2k6MTtzOjU4OiIAU3ltZm9ueVxDb21wb25lbnRcQ2FjaGVcQWRhcHRlclxQcm94  
eUFkYXB0ZXIAc2V0SW5uZXJJdGVtIjtzOjQ6ImV4ZWMiO319Cg%3D%3D%22%2C%22sig_hmac_sha1%22%3A%225f56810b37c66cbcbfb39b6  
e6215c2cb088eff10%22%7D
```

## Working With Documented Gadget Chains
- try to search or look at internet to see if there are any documented exploits are present, that we adapt to attack our target, even if there no dedicated tool for automatically generating serialized object, but we still find documented gadget chains for popular frameworks and adapt them manually.

## Exploiting Ruby deserialization using Documented gadget chain
- we use ruby 2.x Zero DAY Universal RCE by luke janke and encode payload and send through cookie and we exploited sucessfully but for generating payload prefer `online ruby compiler or interpreter `  

# Creating Own Exploit
- when off-the-shelf gadget chains and documented exploits are unsuccessful, we need to create our own exploit.
- for successfully build own gadget chain we need `source code access`.  
	1. 1 st step is study the source code and identify the class that contains the `magic method that is invoked during `deserialization.
	2. Assess the code that this magic method executes to see if it directly does anything dangerous with user-controllable attribute, this worth checking just in case.
	3. if the `magic method` is not exploitable, on its own it can serve as your `kick-off gadget` for gadget chain. study any methods that kick-off gadget invokes. DO any of these , do any of these do something dangerous with data that you control? if not take a closer look at each of the methods that they subsequently invoke and so on.
	4. Repeat this process, keeping track of which values you have access to, until you either reach a dead end or identify a `dangerous sink gadget` into which your controllable data is passed.
	5. once you have work on how to successfully construct a gadget chain within the application code, the next step is to create a serialized object containing your payload.
	6. This is simply a case of studying the class declaration in the source code and creating a valid serialized object with the appropriate values required for your exploit. As we have seen in previous labs, this is relatively simple when working with string-based serialization formats.
- working with binary formats, such as when constructing a java deserialization exploit, can be particularly cunbersome. when making minor changes to an existing object, we might be comfortable working directly with the bytes.
- However when making more significant changes like `passing in a completly new object` this quickly become impractical, it is often much simpler to write your own code in the target language in order to generate and serialize the data ourself.
- when creating our own `gadget chain`, look out for opportunities to use this extra attack surface to trigger secondary vulnerabilities.

## Developing Custom gadget chain for java Deserialization
[lab Portswigger](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-developing-a-custom-gadget-chain-for-java-deserialization)
- we see that `cookie:` header `Session=` have included serialized object, try to tamper with that and see what happen.
- from error we got after tempering with serialized object we know that this is used `java deserialization` 
- from site map we find `/backup` directory contain `productTamplate.java` , this we see `productTemplate.readObject()` method passes the template's `ID` attribute to the `SQL statement`.
- write a java program that serialize our `payload ` in `ID` for sql injection and `base64 encode`it
- `NOTE` for exploitation we need to make same package like what source code of the application have unless it gives us `ClassNotFound Exception`
- they use `PSQL` that is `NOSQL DB` exploit this and get data from the data base
- `NOTE` read errors care fully that given by server for exploiting `PSQL DB` so we exploit easily

## Developing Custom Gadget chain for PHP deserialization
- get php files source code via `~` backup symbol and analyze the source code and try to exploit using that.
- encode object and pass through session and get RCE.


# PHAR Deserialization
- we looked exploiting `deserialization vulnerabilities` where the application/website explicitly deserializes user input, However in `PHP` it is possible to exploit `deserialization ` even if there is no obvious use of the `unserialize()` method.
- PHP provides various `URL-style` wrappers that we use for handling different protocols when accessing file paths.
- one of these protocol is `phar://` wrapper which provides a stream interface for accessing `php archive(.phar)` files.
- PHP documentation revel that `PHAR` manifest files contain `serialized metadata`. 
- if we perform any filesystem operation on a `phar://` stream, this metadata is implicitly `deserialized`, this means `phar://` stream can potential vector for exploiting `insecure deserialization`.
- in this case dangerous filesystem methods i.e `include(), fopen()` are blocked or implement counter-measure for this, but methods such as `file_exists()` which may allowed.
- This technique also requires you to upload the `PHAR` to the server somehow. One approach is to use an image upload functionality, for example. If you are able to create a polyglot file, with a `PHAR` masquerading as a simple `JPG`, you can sometimes bypass the website's validation checks. If you can then force the website to load this polyglot "`JPG`" from a `phar://` stream, any harmful data you inject via the `PHAR` metadata will be deserialized. As the file extension is not checked when PHP reads a stream, it does not matter that the file uses an image extension.
- As long as the class of the object is supported by the website, both the `__wakeup()` and `__destruct()` magic methods can be invoked in this way, allowing you to potentially kick off a gadget chain using this technique.
[portswiggerlab](https://portswigger.net/web-security/deserialization/exploiting/lab-deserialization-using-phar-deserialization-to-deploy-a-custom-gadget-chain)

## exploiting Deserialization using Memory Corruption
- even without use of gadget chains, it is still possible to exploit `insecure deserialization`. 
- if all technique fails there is publicly documented memory corruption vulnerability that can be exploited via `insecure deserialization` and this typically lead to `RCE`.
# protection against insecure deserialization vulnerabilities
 - deserialization of user input should be avoided, unless it is necessary.
 - If possible, you should avoid using generic deserialization features altogether. Serialized data from these methods contains all attributes of the original object, including private fields that potentially contain sensitive information. Instead, you could create your own class-specific serialization methods so that you can at least control which fields are exposed.
 - implement digital signature for checking data integrity before the deserialization take place.
 - 
