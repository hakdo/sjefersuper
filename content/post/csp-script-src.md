---
title: "CSP: protecting against XSS with script-src"
date: 2020-11-24T23:44:55+01:00
draft: false
tags: [xss, csp, supplychain, secheaders]
---
A defence-in-depth strategy for protection against [xss](../xss) is to use 
the `script-src`directive for your [content security policy](../csp).

self
: Only allow scripts from its own domain. This excludes subdomains. 
```
Content-Security-Policy: script-src 'self';
```
unsafe-inline
: If you want to allow inline JavaScript on the page, for example between `script`
tags.

nonce-<base64-value>
: A nonce (number used once) returned from the server on each request. 

<hash-algorithm>-<hash-value>
: A hash of the script, excluding enclosing html tags.

## Recommended practice
To allow inline scripts, it is easy and effective to use the nonce technique. 
Limiting scripts to safe inline scripts and scripts from your own domain is 
a good start.

```
Content-Security-Policy: script-src: 'self' nonce-<my-random-nonce-goes-here>
````

```html
<script nonce=<my-random-nonce-goes-here>>
    alert('donkey')
</script>
```
If you are loading scripts from a different subdomain, you need to add this. 
Also scripts from CDN's and third-parties need to be explicitly added. 

Third-party scripts should also use [subresource integrity](../subresource-integrity) to 
make sure the content cannot be changed after you included it, for example if the 
provider you are loading them from is compromised. 




