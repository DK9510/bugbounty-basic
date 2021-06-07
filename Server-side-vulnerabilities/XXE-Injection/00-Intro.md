## what is XML external Entity Injection.
- it is also known as `XXE` is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. 
- it allows attacker to view files on the application server and to interact with any back-end system that the application it self have access.
- some times attacker leverage `XXE ` to `SSRF` attacks

## How does XXE vulnerabilities arise
- some application use XML format to transmit data between the browser and the server.
- it use standard library or platform API to process the `XML` data on the server.
- `XXE ` arise because the XML specification contains various Potential Dangerous features, and Standard parsers support these feature even if they are not normally used by the application.

# XML entities
## what is XML
- XML stands for `Extensible Markup Language`.
- it is designed for storing and transporting data.
- XML is also uses tree-like structure as HTML uses
- but in XML does not use predefined tags, so tags given name according to user or data.
- it is popular in earlier but now days `JSON` format has widely accepted.

## what are XML Entities
- it is an way of representing an item of data within an XML document.
- various entities are built in to the specification of the XML language.
- example: `&lt; and &gt;` represent the characters `< and >`, these meta characters using to denote `XML tags`, so must generally be represented using their entities when they appear with data.

## what is Document Type Definition.
- XML Document Type Definition`(DTD)` contains declaration that can define the structure of an XML document, the types of data values it contain. and other Items.
- `DTD` is declared with the optional `DOCTYPE` element at the start of the XML document.
- The DTD can be fully self-contained within the document it self( known as `internal DTD` ) or can be loaded from elsewhere (known as `External DTD`) or can by Hybrid of two.

## what are custom Entities
example:
```js
<!DOCTYPE name [<!ENTITY entity_name "entity value" > ] >
```
- in this definition means that any uses of the entity reference `&entity_name;` within the XML document will be replaced by the value of that entity i.e `"entity value"`.

## XML External Entities
- xml external entities are a type of custom entity whose definition is located outside of the `DTD` where they are declared.
- the declaration of an external entity used the `SYSTEM` keyword and must specify a URL which the value of the entity should be loaded.
- **Example:**
```js
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "https://web-dev.com" > ]>
```
we change https protocol to file

```js
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///path/to/file" >]>
```
here we have an `LFI` vulnerability.