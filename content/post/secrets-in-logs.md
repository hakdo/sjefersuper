---
title: "Secrets in Logs"
date: 2020-11-26T15:02:28+01:00
draft: false
tags: [monitoring, secrets, owasp-top-10, vulnerabilities, gdpr]
---
Many projects get logging wrong: 

[OWASP Top 10 - A10: Insufficient logging and monitoring](https://owasp.org/www-project-top-ten/2017/A10_2017-Insufficient_Logging%2526Monitoring)
: No logs available means that monitoring and responding to cyber attacks will be harder. You can't 
respond to problems you can't see. 

[OWASP Top 10 - A3: Sensitive data exposure](https://owasp.org/www-project-top-ten/2017/A3_2017-Sensitive_Data_Exposure)
: Logs containing too much information is a common cause of information disclosure. A common example
is sensitive access tokens landing in server logs, visible to customer support and many others. 

In this post we look at sensitive data exposure in logs. Common ways this can happen include: 

- Sensitive data in URL parameters
- Sensitive data logged during exception handling

## How do we avoid this? 
The first thing we should think about is a threat model as discussed in our 
[secure development lifecycle post](../devlifecycle). 
If we think through *what can go wrong* and then plan how to mitigate that, we are off to a good start. 

We should log events that we anticipate a use for in one of the following cases: 

- Troubleshooting when the application is not working quite right
- Detecting and responding to cyber incidents
- Detecting bottlenecks and improving performance

Keeping sensitive data out of URl's should be a priority as this is exposed many places, including 
in server logs and in the user's browser history. 

If the data you plan to log does not fit any of those use cases, it is a good chance the data is 
not needed. 

Also if you need a particular event to be logged, and that event contains sensitive data that you do 
not need for the use case in question, it is good practice to mask that information. One example 
is of an identifier like an ip address or an access token; you can hash the information to make it 
less sensitive. 

{{<warning>}}
Exposing personal data that can result in a breach of the privacy rights of individuals may be 
considered a breach of privacy legislation such as the GDPR. If this is needed, you should perform 
a legitimate interest assessment first, involving your organization's privacy specialists.  
{{</warning>}}

## Allowing sensitive data for debugging and testing
If you are debugging or investigating an unexpected event, turning on detailed debug logging 
including the sensitive data in question can be a good solution. This should be done using 
synthetic test data in an environment that is separate to the production environment. This way, 
the data exposed will not have any impact for the application, its data and its users. 