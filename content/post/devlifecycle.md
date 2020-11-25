---
title: "Development Lifecycle"
date: 2020-11-25T22:08:18+01:00
draft: false
tags: [sdlc, vulnerabilities, threatmodeling]
---
Creating a software that is safe to use is no easy task. We 
will look at some activities in different phases. The backdrop is that 
vulnerabilities in software have two different sources; 1) flaws in business 
logic and architecture, and 2) bugs. 

- Planning
- Development
- Testing and validation
- Monitoring
- Issue prioritisation

## Planning
During planning we should think not only about how the application will fulfill its 
purpose, but also how we can avoid problems with respect to reliability, performance 
and security. The most important part for security is to create a *threat model*. 

A threat model is a description of how threat actors can potentially attack the 
application to achieve some objective they are not supposed to be able of doing. 
There are several ways to do this, of the more well-known approaches are STRIDE, 
Protection Poker, and Elevation of Privilege. No matter what method we choose, 
what we want to know is how the application can be attacked, so that we can 
plan security requirements to stop that attack from happening, or the very least
discover that it has happened. 
This is effective for discovering vulnerabilities that are not caused by bugs, 
but rather by flaws in business logic.

## Example: threat modeling
{{<note>}}
Forgot password: the user enters the registered email address. The user is then 
emailed a link with an embedded user ID to reset their password. 
{{</note>}}

Potential attacks: 
- Attacker gets access to the link with the embedded user ID after hacking the user's email account and uses it to reset the password
- The attacker registers an account to discover the structure of the password reset link.
The attacker then systematically resets the password for every user by iterating over 
the user ID

How are these attacks made possible? These are the vulnerabilities we the attacker 
is exploiting. 

- The user ID exposed in the link is predictable, allowing access to other people's user ID's. This is a case of "Insecure Direct Object Reference". 
- The link in the email sent when a user wants to reset a password can be used multiple times
- The link does not expire after a certain amount of time
- The user is not notified when a password is reset
- The link is captured by an attacker with a monkey-in-the-middle (MitM) position


## Developing security requirements
We should use the vulnerabilties identified in the threat model to develop 
testable security requirements. ¨

Let us do that for the password reset link we discussed in the example above. 
We should test that these requirements are fulfilled before the application is allowed to 
be deployed to production. 

Requirements: 
- The user ID should be repalced with a random, non-guessable token. Sensitive 
URL parameters should not be guessable.
- The link for password reset should only be possible to use once
- THe link should expire after some time (1 hour)
- The user should receive a notification by email or otherwise when a password is changed
- The email should only be transferred over an encrypted connection (TLS)

If all these requirements are fulfilled, it will not be easy for an attacker to use 
the functionality to gain access to someone's user account.

## Writing secure code
When we want secure code in our application, we need to learn some basic techniques 
that can stop common attacks. 

- Don't trust user input: always sanitize and validate input with respect to size, type, structure and content
- Don't trust that input is coming from the right sender. Always authenticate 
network communications
- Don't assume resource are always available, control resource consumption
- Be conscious of vulnerabilities in third-party software libraries
- Expect exceptions, and handle them
- Log application events that are important. Sensitive transactions, 
undesired behavior, errors and exceptions
- Separate your code into modules that can be unit tested

## Test and validate
Testing should be part of the workflow. As a minimum the following 3 levels of testing 
should be used: 

1. Check dependencies for known vulnerabilities (can be automated with software)
2. Check the source code with a static analyzer (also automated)
3. Write unit tests covering every security requirement that is based on the threat model

Ideally you should not allow any known vulnerabilities to go to production, but at least make this the basic rule of enagement: 
{{<warning>}}
Do not allow vulnerabilities that are easy to discover and exploit in production.
{{</warning>}}

## Monitoring in production
When you have your application in production, you need to monitor it. If 
somebody is trying to attack it, someone should notice and do something about it 
if we are at risk of violating the security expectations of users or app owners. 
In addition, we need to monitor for security vulnerabilities that are discovered 
in the software after it went into production. 

*Discovering attacks*: to discover attacks we need to log attempts of behaviors we 
do not approve of. A good threat model is thus an excellent tool for planning 
what events to log from the application itself. Vulnerabilities we are only 
able to detect, and not prevent, should be logged with high priority, and preferably 
generate an alert to someone who can react to it. 

*Discovering vulnrabliities*: vulnerabilities in a production applicaton can be in 
our code, in the infrastructure and environment we depend on, or in third-party 
dependencies in our code (external libraries). 

For vulnerabilities in our app that 
are discovered by security researchers, we should provide an easy way to report 
them to us. A good idea is to add a `.well-known/security.txt` file that explains
how to report issues.

We also need to check for vulnerabilities in third-party libraries regularly. Several 
code repositories can now monitor this for us and alert us to such problems. 
For example, 
[Github](https://github.blog/2020-06-01-keep-all-your-packages-up-to-date-with-dependabot/) 
can both detect and alert us to it, and provide a fix in a 
pull request. The solution we use should at least be automated, so it is not forgotten. 

## Prioritising issues
When we discover vulnerabiltiies, fixing them is important. It can be easy to think 
that we will only fix the "important" things, but unfortunately, allowing a lot of 
"low-risk" vulnerabilities can lead to risk creep. Several of them may be chained together 
by an atatcker to create higher impact, and fixing them later can become very costly. 

A good strategy can be to prioritise any vulnerabilities that can lead to unacceptable 
impact with limited cost for the attacker should be fixed as soon as possible, whereas 
less critical issues are planned into the normal development as "maintenance work". 

The konwn vulnerabilities in the application should be monitored. If the technical debt 
related to security vulnerabilities is increasing with time, we have a problem. 
Technical debt carries a very high interest, and if it is not paid off according to schedule 
it will grow beyond our control. 

