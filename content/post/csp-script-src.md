---
title: "Content Security Policy: the script-src directive"
date: 2020-11-24T23:44:55+01:00
draft: false
tags: [xss, csp, script, secheaders]
---
One of the most common web vulnerabilities is cross-site scripting. This 
is an injection vulnerability where an attacker is able to inject 
JavaScript into a web page through a user input or an exposed parameter. 
Typical injection points are form fields and URL parameters. There are 3 different 
types of cross-site scripting (XSS): 

Reflected XSS
: user input is reflected directly back at the user

Persistent XSS
: user input is stored in a database and output somewhere in the user interface.

DOM based XSS: 
: user input is parsed and reflected directly in the DOM

# What is a content security policy (CSP)?
A [content security policy](/csp) is a security header that allows web site owners 
to protect its users from client-side attacks by controlling browser security 
features. It replaces many of the older "security headers". 



