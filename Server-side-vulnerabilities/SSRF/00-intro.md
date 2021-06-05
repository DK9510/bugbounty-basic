# Server side request Forgery(ssrf)

## what is SSRF
- it is web security vulnerability that allows an attacker to induce the server-side application to make HTTP request to an arbitrary domain of the attacker's choosing.
- it cause the server to make connection to internal-only service within org's infrastructure.
- or they able to force the server to connect to arbitrary external system, for leaking sensitive data, such as Authorization creds.

## Impact of SSRF attacks
- the common impact is unauthorized access to data with in organization.
- in some situation it might allow attacker to perform RCE.

## Common SSRF Attacks
- the attacker induce the application to make an HTTP request back to the server that is hosting the application via its loopback network interface.(127.0.0.1/localhost)


`Example:`
- a store web fetch that item is in stock or not
```JS
POST /product/stock HTTP/1.0  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 118  
  
stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1
```
here front end request to Back end using REST API to query database for stock by the server it self

if we modify that request to 
```js
POST /product/stock HTTP/1.0  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 118  
  
stockApi=http://localhost/admin
```
we have make request to `/admin` panel using using server so we have request locally .
it can bypass normal access control and expose admin panel to attacker.

### Basic SSRF against the local server
- simple change the parameter that request to fetch stock or internal request 
- `StockAPI=http://localhost/admin`

-   The access-control check might be implemented in a different component that sits in front of the application server. When a connection is made back to the server itself, the check is bypassed.
-   For disaster recovery purposes, the application might allow administrative access without logging in, to any user coming from the local machine. This provides a way for an administrator to recover the system in the event they lose their credentials. The assumption here is that only a fully trusted user would be coming directly from the server itself.
-   The administrative interface might be listening on a different port number than the main application, and so might not be reachable directly by users.

