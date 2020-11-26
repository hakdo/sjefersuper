---
title: "CSP Frame Ancestors"
date: 2020-11-26T23:48:45+01:00
draft: false
tags: [csp, secheaders]
---
A [content security policy](../csp) can be used to protect against [clickjacking](../clickjacking)
on modern browsers. 
This is done by controlling where your website can put in an iframe. If you want to allow framing 
on pages on the same origin, you can use the 'self' specification. A typical practice for pages meant 
to be framed is to include 'self', and explicitly list domains that are allowed to frame the page: 

```
Content-Security-Policy: frame-ancestors 'self' https://www.mycustomer.com
```

If you want to completely block framing, the following directive will do that: 

```
Content-Security-Policy: frame-ancestors 'none'
```

Also, remember that Internet Explorer still exists and does not support CSP. You should therefore
consider using the 'x-frame-options' header in addition to a CSP in your defence, since IE supports 
this header (and was also the browser introducing it, in Internet Explorer 8).