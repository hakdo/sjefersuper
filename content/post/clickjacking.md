---
title: "Clickjacking"
date: 2020-11-26T23:25:53+01:00
draft: false
tags: [vulnerabilities]
---

## What is the problem?
Clickjacking is a user interface based attack, where a user is clicking content on a hidden webpage, 
by tricking them to try to click something on a decoy webpage. If the hidden page is a page the 
user is logged in to, the attacker can trick the user into performing actions he or she is unaware of.

This can lead to: 

- Making a payment without knowing about it
- Clicking a link to somewhere
- Downloads that are not what they seem

Here is one example of a clickjacking attack where the webpage of the World Health Organization is used 
as the decoy page but where an invisible iframe is put on top of the WHO logo to trick the user into 
clicking a link.

{{<youtube ZPnHS8hxU7o>}}

## How can you defend against clickjacking attacks?
All web pages that allow framing can be abused in clickjacking. 
The primary defence against this 
consists of limiting where that page can be framed. The 'modern' browser based defence is the use 
of a [Content Security Policy](../csp). It is also possible to use the `X-Frame-Options`header.
This header is unfortunately not implemented consistently across all browsers but can still be an 
effective defence against clickjacking, especially when combined with other techniques. 

Another strategy that can be used is frame busting scripts. These are JavaScript routines included to 
detect and interrupt framing. They are often possible ot bypass for attackers but can provide an extra 
layer of defence. 

{{<note>}}
The primary defence against clickjacking should be the use of a CSP with the 
frame-ancestors directive.
{{</note>}}

See how to configure [frame-ancestors](../csp-frame-ancestors).

