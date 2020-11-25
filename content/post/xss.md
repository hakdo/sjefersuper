---
title: "XSS: Cross-site scripting"
date: 2020-11-24T23:30:53+01:00
draft: false
tags: [xss, owasp-top-10]
---
One of the most common web vulnerabilities is cross-site scripting. It is number 7 on 
OWASP TOp 10 [(2017)](https://owasp.org/www-project-top-ten/2017/A7_2017-Cross-Site_Scripting_(XSS)).
This is an injection vulnerability where an attacker is able to inject 
JavaScript into a web page through a user input or an exposed parameter. 
Typical injection points are form fields and URL parameters. There are 3 different 
types of cross-site scripting (XSS): 

Reflected XSS
: user input is reflected directly back at the user

Persistent XSS
: user input is stored in a database and output somewhere in the user interface.

DOM based XSS: 
: user input is parsed and reflected directly in the DOM

Such attacks can, alone or in combination with others, lead to account hijacking, 
content injection. The persistent type of XSS is the most severe. 

## How to protect
1. If you can use a framework that automatically protect against XSS, this is 
a good starting point as most cases will be covered. 
2. Make sure to escape HTTP request data based on the context of the output where 
it is used, to avoid active content executing in the user's context. 
3. Use a [content security policy (CSP)](../csp) with the script-src directive set to control how JavaScript can be executed in the 
application.