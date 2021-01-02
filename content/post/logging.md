---
title: "Logging"
date: 2020-12-20T08:01:01+01:00
draft: false
---
A popular expression within security is "prevention is ideal but detection is a must". 
If you can detect what attackers are up to, whether they succeed in breaching your app 
or not, you can make it more difficult for them, or at least limit the damage they 
can do. We need logs to detect attacks. 

{{<figure 
    alt="Logs on fire"
    src="/images/logging.jpg"
    caption="In security we need logs to put out the fire. Photo by [Elijah Hiett](https://unsplash.com/@elijahdhiett?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) on Unsplash" >}}

## Server logs
A typical web server (Apache, nginx, etc) will generate some standard logs for us. 
They will typically contain error logs and access logs. These provide a good start 
by capturing essential information such as the ip address requesting a resource, 
http response codes and user agent strings. Often they are not enough for effective 
incident response and defenders need to rely on application level logs. 
It is thus important that the developer takes expected attacks into account and 
creates detection opportunities. The tool you can use to identify such scenarios 
is *threat modeling*, an important part of a [secure development lifecycle](/devlifecycle).

## Context matters
Logs will be used for investigations; someone is looking at your logs to try to find out 
what has happened. It is therefore important to provide context that will help the 
investigation. Think of the difference between these log lines: 

`User transaction failed`

and 

`WARNING: User transaction failed: credit card marked in deny list for user ID xxxx`

Say your application performs credit card transactions and checks a deny list for 
every transaction, and you se a single user performing a high number of transactions 
using different credt cards, where half of them give the error above. What would this 
be an indication of? It may be time to call the police. 

## Security logging at Layer 7
There are many reasons for creating application logs but our concern is that of 
security. We want to protect the confidentiality, integrity and availability 
of our services. There are 3 basic questions we need help from application logs 
to answer: 

1. Who is authenticated from where in the application and when is it happening?
2. What application activities are taking place that are suspicious or unusual?
3. Is the application behaving as expected?

## Keeping secrets secret
Logs are not good at keeping secrets. Many people may have access to them, and they 
are parsed in systems that you may not have control over. Because of this, it is a 
golden rule to keep secrets and sensitive data like passwords, credit card numbers and 
passport numbers out of your logs. 

Sometimes the secret itself can be helpful in identifying undesired patterns. In such 
cases, we can use a hash of the secret or another making technique to avoid 
spilling the secret in the log. 

## An example: password sprays
A common attack on password protected applications is *password spraying*. This is a 
technique where the attacker tries to log in to many user accounts, one by one, using the 
same password. It is a brute-force attack variant that is more likely to escape 
detection than trying many passwords in a row for a single account. If a security 
analyst sees the following log events, they will think that someone is trying to gain 
access to a user account by brute-force: 

```
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Failed login for user xxx - wrong password
Successful login for user xxx
```

If instead that attacker is using the same password and trying one account after 
the other, it is less likely to stand out or create automated alerts. Let us try 
to anticipate the behavior of the attacker. If this is a skilled attacker 
they will know that it is common to use "impossible travel" to detect attacks. An 
impossible travel detection is done by comparing the geolocation of the ip address
of a login request with the previous logins of that user. If the same user is seemingly
logging in from London in the morning and Perth just after lunch, this is 
probably not what is actually happening (planes are not that fast). So the attacker, 
anticipating this, now uses a VPN to try to login from a local ip address for 
each user. This means that typical security detections done by

- Many login attempts from the same ip address for a single user
- Impossible travel

will not work here. Further, if the attacker switches ip address between each 
login attempt, or at least only doing a few attempts from each ip address, detection 
will be more difficult. 

Back to our developer and defender perspective. We are expecting clever spraying attacks. 
Repeated behavior is a typical artefact of malicious activity. In this case the repeated
behavior is the password itself, but since logging the password itself could leak it 
we cannot do that. Even if we only log failed passwords, this is not a good idea because
it would also log "almost correct" passwords, which would make the actual password 
easy to guess for someone with access to the logs. 

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">&quot;Nobody gets hacked. To get hacked you need somebody with 197 IQ and he needs about 15 percent of your password.&quot;<a href="https://t.co/6aR8yU2MVg">pic.twitter.com/6aR8yU2MVg</a></p>&mdash; Martin (@mshelton) <a href="https://twitter.com/mshelton/status/1318303047647309824?ref_src=twsrc%5Etfw">October 19, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Hence: we log hashes (but don't use 
the same hashing algorithm as the actual password hash in your authenticaiton database).
Let's say we have this pattern: 

```
Failed login for user donald - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user mickey - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user scrooge - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user dolly - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user minney - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user pluto - wrong password (a56e87677544c2ead932668270202b98)
Failed login for user goofey - wrong password (a56e87677544c2ead932668270202b98)
Successful login for user rockerduck - password (a56e87677544c2ead932668270202b98)

```
In this case we have an opportunity to detect spraying attacks based on the 
hashed version of the password.

## Are there some rules of thumb? 
Yes, we can create some rules of thumb. Let's call them our 10 commandments of 
layer 7 security logging!

1. Make it easy to tune verbosity of logging up and down according to neeed
2. Do create threat models and use them to figure out what to log
3. Make sure the logs are sent tot a separate storage system that is not reachable
from a compromised application (to avoid tampering)
4. Log all authentication attempts (successful or not), authorization changes, 
password resets and critical user account events (e.g. account lockout or account 
deletion)
5. Use a standard log formats with descriptive log levels
6. Provide enough context for the investigator to understand what is going on
7. Use a logger that will help with formatting and allows multiple sinks
8. Cooperate with security analysts to help create logs that can be used in automated 
detections
9. Also log actions performed as part of automated remediation (e.g. account lock-out 
due to too many failed login attempts)
10. Don't log events that you do not have a clear use case for. Too much data is as bad 
as not enough data!

