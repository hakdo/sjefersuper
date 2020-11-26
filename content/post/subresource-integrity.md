---
title: "Subresource Integrity"
date: 2020-11-24T23:30:53+01:00
draft: false
tags: [supplychain]
---

Subresource integrity is a standard that helps make sure that 
third party content allowed in a web page is the same content 
the developer intended to put there. You can see if someone uses it 
by looking for the *integrity* attribute on external content. 

```html
<link rel="text/css" href="some-url-to-a-third-party" integrity=<a-hash-goes-here>>
```

If the hash of the file loaded is different from the one specified, the content 
will not load. 