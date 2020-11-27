---
title: "Abuse Stories"
date: 2020-11-27T19:34:11+01:00
draft: false
tags: [threatmodeling]
---
Developers often use *user stories* in planning of software features. User stories 
are descriptions of how a feature should work from an end-user's perspective, typically 
in an informal way. This has several benefits; 

- It helps keep focus on the user instead of technology
- It inspires collaboration and creativity
- It makes it easier to integrate user experience in the software design process (UX)

In security, there is a similar concept, called *abuse stories*. An abuse story is a 
description of an outcome achieved by an attacker by exploiting vulnerabilities or features 
in ways the software designer did not intend. Abuse stories bring the same benefits to security 
engineering as user stories brings to software design. Abuse stories can be used together with a 
more formal threat modeling approach like STRIDE, or instead of it in cases where rigor is 
less important. 

## Abuse stories for an authentication system
Consider a typical authentication system where a user logs in to some restricted resource using 
a username and a password. Perhaps the restricted resource is an online store with stored 
payment details in user profiles, so that you can order items and pay with an existing credit card 
on file - if you can only get in. You try some basic testing by creating a user account and 
testing various features as a normal user. 
The application does not have account lockouts. It also appears that there is not any form of rate 
limiting on its authentication system. 
The password policy is a minimum length of 5 characters. 

You are now the hacker. What do you want to do with the information you have gathered?
Abuse stories can be as simple as: 

{{<ticks>}}
* Main abuse story: I want to break into user accounts to order stuff for free that I can then sell 
on online market places. 
{{</ticks>}}


### Sub-stories: how the attacker plans to accomplish that goal in the main story
Phishing
: I will gain access to the user account through phishing for the credentials and then logging in

Credential stuffing
: I will reuse credential dumps from the dark web and see if I can get access to a user account

Spray
: I will enumerate the users and then run a password-spraying attack to gain access to user accounts

## Building protection in
Obviously if it is your online store, you don't want angry customers and the data protection 
authorities on your neck. What should you do? In creating the abuse stories we have thought 
about how to exploit vulnerabilities to make them possible. What if we do the following? 

- Add a strong password policy
- Blacklist common and known leaked passwords
- Lock out accounts after 5 wrong attempts
- Send an email to the user whenever there is a login from an unknown ip address
- Add the option of using two-factor authentication
- Design elements nudging the user to check the browser address bar to see if the URL is correct
- Change the payments to require two-factor for the transaction 
(which is required by the PSD2 directive in Europe and implemented in payment standards)

If we build these things into our login process, the abuse stories above are unlikely to be 
successful for our hacking friend. Using abuse stories like this can be sufficient to identify 
security requirements that make attacks mode difficult. The process is a 3-step process: 

1. Describe how the application works
2. Pretend you are the criminal and articulate what you want to achieve (order free stuff)
3. Develop sub-stories (how to come in a position to order free stuff)
4. Plan features that make it hard to succeed with the sub-stories, and thus also the main story (ordering that free stuff)





