# Directory Traversal
## what is Directory Traversal/Path Traversal
* it is also known as `path-traversal` vulnerability that include application `data,code,credentials,sensitive os files` 
* if some of these files have write permission, allowing attacker to modify the content of the file and take full control of the server.

## reading arbitrary files via directory traversal 
```js
<img src="/loadImage?filename=218.png">
```
in this request send to `?filename=` parameter which we can exploit by traversing path like
```js
https://vuln-web.com/Loadimage?filename=../../../../etc/passwd(or any sysfiles)
```

## prevention
* to avoid passing user-supplied input to filesystem APIs altogether. Many application functions that do this can be rewritten to deliver the same behavior in a safer way.

# exploitation
* Many applications that place user input into file paths implement some kind of defense against path traversal attacks, and these can often be circumvented.
 * if application blocks or strips directory traversal sequence there is many bypassing technique is available
 * we might directly access absolute path `file=/etc/passwd` with out using any traversal.
 * You might be able to use various non-standard encoding, such as `..%c0%af` or `..%252f`, to bypass the input filter.
* [filter-bypass-technique](https://code.google.com/archive/p/teenage-mutant-ninja-turtles/wikis/AdvancedObfuscationPathtraversal.wiki)
* might user `....//`or `....\\`if one `..../ or ....\ ` is blocked than after removing them we traverse it.
* If an application requires that the user-supplied filename must start with the expected base folder, such as `/var/www/images`, then it might be possible to include the required base folder followed by suitable traversal sequences. For example:`filename=/var/www/images/../../../etc/passwd`
* If an application requires that the user-supplied filename must end with an expected file extension, such as `.png`, then it might be possible to use a null byte to effectively terminate the file path before the required extension. For example:`filename=../../../etc/passwd%00.png`