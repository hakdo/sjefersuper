---
title: "CSP: Content Security Policy"
date: 2020-11-25T08:22:50+01:00
draft: false
tags: [csp, secheaders]
---

A content security policy (CSP) is a modern security header that allows you 
to tell the browser to use certain built-in security features. The web server should 
return the Content-Security-Policy HTTP header. As alternative you may use 
`<meta>` tags.

{{<warning>}}
A content security policy should be viewed as a defence-in-depth mechanism and 
be used in addition to other defence mechanisms.
{{</warning>}}

To use a CSP, set the web server to run this header:

```
Content-Security-Policy: policy-goes-here
```

# Threats and relevant directives
| Threat | Relevant directive |
|--------|--------------------|
|[xss](../xss)|[script-src](../csp-script-src)|
|clickjacking |frame-ancestors|

# Testing your policy
Before enabling a CSP directive, you should its effect on the site. This can 
be achieved by setting the content security policy using the *report only* header
like this: 

```
Content-Security-Policy-Report-Only: policy-goes-here
```
This is important to do, especially if you load a lot of third-party content. 
It is likely that introducing a strict CSP on an existing site will break things, 
and that you will need to apply some fixes before it works smoothly.

## Turning on reporting
There is no point in turning on reporting unless you are planning to use those 
reports for something. The CSP directives work without reporting.

You can turn on reports of policy violations. Then you need to supply a directive that 
tells the browser where to send those reports. The traditional way of doing this is 
to provide an endpoint that will receive the reports: `report-uri`. This is now 
deprecated and the official recommendation is to use the `report-to`directive with a 
JSON object explaning where to send those reports. The `report-to` directive is not 
supported yet in most browsers, so if you want reporting, you should probably 
configure both. You can read more about this at https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri. 

# Other sites with more
- Documentation on use and compatibility: [CSP documentation on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- Bypasses and loopholes: [Portswigger Web Security Academy](https://portswigger.net/web-security/cross-site-scripting/content-security-policy)

