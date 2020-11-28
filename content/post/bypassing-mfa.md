---
title: "Bypassing Mfa"
date: 2020-11-28T12:36:15+01:00
draft: true
tags: [vulnerabilities, owasp-top-10]
---
Multi-factor authentication (MFA), to-factor authentication, or perhaps two-step verification? 
They are all terms used to describe an authentication concept meant to be stronger than just a 
username and a password. Whereas the password is "something you know", the second factor should be 
proof of "something you have". 


{{<figure 
    alt="A phone is something you have"
    src="/images/phone.jpg"
    caption="A phone can be something you have in a MFA concept" >}}


In other words; two different ways of authenticating. This is sound
practice and conforms to an old idea in barrier engineering as used in the chemical process industries, 
nuclear power plants and aviation; 

{{<note>}}
For every sufficiently high risk situation, there should be two independent barriers that 
are able to stop the event propagation on their own and that are independent of each other. 
{{</note>}}

If we are able to create such barriers, it will be very unlikely that this event is going to occur, 
or at least that we will slow down the propagation of the event enough to intervene to make the 
consequences less severe. This is true for nuclear power plants as much as it is true for the 
login to your company secrets. Two good questions we need to ask is: "how independent is good enough?", 
and "how do we ensure the barriers themselves have sufficient reliability to provide the desired 
protection?". These are common questions to ask in the design of safety critical systems, but in IT 
it is much less common. We should start asking those questions more often. 

## A typical OTP-based two-factor authentication concept {#abuse}
Let us consider a system where a user first logs in with username and password, and is then prompted 
for a one-time password (OTP), typically a 6-digit code provided by an app on the user's phone, 
or sent to the user's phone by SMS.  An attacker has managed to obtain a valid username and password, 
perhaps with one of the techniques described in [Abuse Stories](../abuse-stories). How could this 
attacker get past our OTP-prompt, designed to require control of the victim's cell phone? Here are some 
options: 

Phishing
: Set up a flow that looks like the real login flow, and also capture the OTP to log in as the 
victim. Setting up a two-step phishing site is not much more difficult than only stealing the password, 
but it will have to be reused at once. This can be done manually but immediately by the attacker, or 
the attacker could set up an automated service where the backend of the phishing application makes 
requests directly to the target server, practically gaining a monkey-in-the-middle (MitM) position.

Steal the victim's phone
: The attacker could of course steal the victim's phone. If SMS is used for the OTP, the message 
can often be read on the screen of a locked phone, and some OTP apps allow that too. If not, the attacker
would have to steal the phone while unlocked, and if the victim is on the other side of the planet this 
is impractical anyway. 

SIM swap attacks
: Media has written a lot about this: the attacker uses social engineering to trick the phone company
to transfer the victim's number to a new SIM card held by the attacker. It is unfortunately easier than 
it should be, but for most accounts this is a less likely scenario.

Obtaining the OTP seed to reproduce codes at will
: If the attacker can get access to the seed used to generate OTP codes for the user account they want to attack, they can reproduce the codes by using the same OTP generator code. It is thus very 
important to protect the OTP seeds.

Brute-force attacks on the OTP validation step
: Most OTP based MFA applications allow about 30 seconds for each code ot be used, some a bit longer. 
If there is no mechanism stopping it, the attacker can just attempt to loop through all of them within 
that time and in this way break in. 

SQL Injection attacks
: If the code checking the OTP in the backend is vulnerable to SQL injection, the attacker may be able 
to bypass the whole OTP check in the same manner as SQL injection attacks can sometimes 
bypass password forms.

These can all be classified as 
[broken authentication](https://owasp.org/www-project-top-ten/2017/A2_2017-Broken_Authentication), 
a good number 2 of vulnerabilities on the OWASP Top 10 list.

## How do we protect against such attacks? 
The best way to protect against attacks like this can for many use cases be to use 
a third-party authentication system. If you need to build your own MFA system, 
think about expected attack patterns and build in defences accordingly. Here are some suggestions 
based on the [list of abuse stories above]({{< relref  "bypassing-mfa.md#abuse" >}}).

| Attack type | Protection |
|-------------|------------|
| Phishing    | Send alert email to user on unusual logins, add UI elements to nudge the user to check the URL in the address bar. 
| SIM Swaps   | If the app is a high-value target for skilled hackers, consider not offering SMS based OTP.
| Brute-force attacks | Apply rate-limiting behavior on OTP trials. Consider use of lockouts, or a ReCAPTCHA.
| SQL injection | Sanitize inputs, use prepared statements.|
| Bypass MFA by using a legacy protocol | Make sure you do not expose other authentication protocols that do not trigger the MFA process.

The vulnerabilities we have discussed here are "failure modes" for the secondary barrier. This *could* 
mean that the barrier is not reliable enough to be used to provide the desired integrity. With 
proper hardening, however, we can significantly improve that reliability. In the phishing case, 
we have the same "failure mode" for both password and OTP. Sometimes that is hard to avoid, 
but it also means we cannot see the two ways of authenticating as completely independent barriers. 
They have a common cause failure (CCF). 

The key take-away here is 
not that MFA is not a good idea (it is), it is that for two independent barriers against a high-risk
scenario to be effective they must: 

- Be able of doing the job on their own and work in independent ways, as far as possible
- Be reliable enough on their own to provide the desired risk reduction effect

# Some more reading on MFA bypasses this elsewhere
[Write-up of a brute-force attack on MFA (Medium post)](https://medium.com/devopsenthusiasm/bugbounty-how-i-cracked-2fa-two-factor-authentication-with-simple-factor-brute-force-a1c0f3a2f1b4)

[Example of phishing kit bypassing MFA (ZDNet)](https://www.zdnet.com/article/new-tool-automates-phishing-attacks-that-bypass-2fa/)

[Story about SIM swapping attacks on users of crypto exchanges (ZDNet)](https://www.zdnet.com/article/wave-of-sim-swapping-attacks-hit-us-cryptocurrency-users/)

[Microsoft docs explaining how legacy protocols don't support MFA](https://docs.microsoft.com/en-us/azure/active-directory/conditional-access/block-legacy-authentication)


