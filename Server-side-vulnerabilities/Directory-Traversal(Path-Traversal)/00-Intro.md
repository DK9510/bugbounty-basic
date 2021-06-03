## what is Directory Traversal
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
* 