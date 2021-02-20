---
title: "Threatmodeling"
date: 2021-02-20T09:01:39+01:00
draft: false
tags: [threatmodeling, vulnerabilities]
---
## tl;dr:
Threat modeling helps you avoid business logic flaws that make your system vulnerable. Here's what you should do. 

1. Create description of how something works, e.g. a user story or data flow diagram.
2. Brainstorm ways an attacker can achieve outcomes you did not intend. Works best as a group activity. Remember coffee and snacks and put your evil hat on.
3. Figure out ways to stop those attacks from happening. Engineering hat on. Or security guard.
4. Create test requirements that will verify the attacks don't work anymore after implementing the security measures you thought of. 

## What is threat modeling and why do we need it?
There is not such thing as unhackable technology. When we design a software, we 
know it will have bugs: we don't have the capability of designing perfect things. 
That means, the things we build will have vulnerabilities. We want to minimize the 
number of vulnerabilities. 

There are two types of vulnerabilities, and they are approximately equally common.

- Bugs: vulnerabilities introduced through programming errors
- Business logic flaws: introduced because we have flaws in our design

We can fight bugs using things like static analysis but that can't help us detect a poor plan
for implementing the business logic. This is where threat modeling comes into play. We want to 
find potential vulnerabilities, and then build ways in to mitigate exploitation attempts. There 
are a few ways to mitigate exploitation: 

- Change the design to remove the vulnerability completely (can't always be done)
- Add security controls in the software to remove the vulnerability or make exploiting it more difficult (usually possible)
- Add detection of exploitation attempts to allow a security team to handle it (less effective, likely to fail)
- Accept the risk and do nothing (easy to do but can cost you down the road)

Threat modeling is simply the activity of looking at your system design and figuring out 
what can be abused by someone to achieve an outcome in your system you as a designer did not intend. 

There are many formalisms for doing threat modeling, such as Microsoft's STRIDE methodology developed 
in the 1990's. This can be helpful but often it is enough to look at your documentation and think about 
how an attacker could try to abuse this. Let's do that with a small example. 

## How to create a threat model
{{<note>}}
We are building an app for our new startup, QuickRich. The app uses banking API's to pull 
information from bank accounts and stock brokers into a single app, and will automatically 
suggest investments and savings actions to take to help users get rich.
{{</note>}}

Before we can start we need a reasonable description of the system we are building. 
Two useful description types for building threat models include

- User story diagrams
- Data flow diagrams

Let us focus on a single user story in our QuickRich app. 

> "As a user I want ot authenticate to the app and get access to my data."

Now we want to find ways this can go wrong - so let's just ask the question "what is the worst that could happen?"
and try to answer that from different stakeholder perspectives.

| Stakeholder | What is the worst that could happen? |
|-------------|--------------------------------------|
| User        | Someone else gets access to my accounts and can steal my money |
| QuickRich CEO | Someone gets access to all the user accounts and we get a huge GDPR fine, get sued and I go bankrupt |
| IT boss@QuickRich | Someone tries to or gets access to people's accounts and we don't even detect it |

So, from this we see that the primary abuse case we need to take into account is someone breaking into a user account, 
and that prevention is ideal but detection is a must. Our freelance developer has suggested that users can log in with 
username and password, and with a one-time code sent on SMS to get in. 

{{<figure 
    alt="User story and data flow diagram"
    src="/images/threatmodeldia.jpeg"
    caption="Developing a threat model at the user story level allows you to be earlier with finding problems, making it easier and cheaper to fix them." >}}

A good way to perform the threat modeling is as a brain storming exercise. Having a few "hooks" to lean on of typical attacks that could 
happen can be helpful. The following quick list of things to consider can work as a start: 

- Social engineering & phishing
- Brute-force attacks
- Web app injection attacks (XSS, SQL injection, etc.)
- Abuse of authorization weaknesses
- Denial of service
- Backdoors
- Evil insiders

Then we put ourselves in the shows of the attacker. Here are some goals the attacker could have: 

- Break into a single user account
- Gain access to all user accounts

Let us start with the task of gaining control over a single user account. Based on the user story, here are some things 
to try as an attacker. We assume we have managed to enumerate users of the app, so we have a list of usernames. 

- Credential stuffing: try leaked passwords 
- Brute-force attack on password form (dictionary attack)
- Password spray (slow attack with same password across many user accounts)
- Phishing a user for the password using a fake login page

OK, so the attack has managed to find the right password. Now they need to get that SMS. How can we do that as an attacker? 

- SIM swap attack: trick the phone company to transfer the victim's phone number to their own SIM card. 
[Yes, this really is a problem (wired.com)](https://www.wired.com/story/sim-swap-attack-defend-phone/). 
- Brute-force attack on the one-time code?

{{<note>}}
We managed to identify a number of possible threats to the authentication process, without even looking at the 
data flow diagram. We have found that SMS is a weakness by itself, and that the password part of the process 
needs protection against some types of attacks. This allows us to make security decision even before creating 
detailed designs!
{{</note>}}

Moving on to the data flow diagram, we would be able to identify more attack vectors. If an attacker 
has looked at the application and found out there is a frontend component, a backend API, a database, 
and an SMS gateway, the attacker has many more things to try. Things can go wrong inside each "node", 
and they can go wrong during data transfers. 

### Attacking the web form itself
- Are there any cross-site scripting vulnerabilities allowing us to inject javascript code to steal passwords, tokens or cookies?
- Are there SQL injection vulnerabilities allowing us to download the entire user database, with password hashes? 

### Data transfer attacks on the API
- Can we get a monkey-in-the-middle position and steal passwords and tokens from the network? 

### Database attacks
- Is the database available directly on the internet and can we brute-force or find the connection string?

### SMS gateway attacks
- Is the SMS Gateway API secure? Can we impersonate the API server and get the SMS functionality working for a phishing site?

As we see, we find other vulnerabilities in the data flow diagram than with the user story diagram as the basis 
of our discussion. Some of these attacks are easy to avoid by using good coding practices, such as input validation, 
prepared statements and using HTTPS on all data transfers, whereas others point to API authentication secrets for the 
SMS gateway, something we need to know how the gateway setup works to consider. However, if we look at the threats we found 
based on user stories and decide not to use SMS at all, that vulnerability is of course removed (but there may be 
vulnerabilities in the solution we choose to use instead as well).

This demonstrates and important point: your threat model must evolve as your design becomes more mature. A good idea is 
to review your threat model when selecting user stories, and when you have a data flow diagram in place. WHen the data flow 
diagram changes, you should check whether this has an effect on the threat model: does it remove possible attacks, or allow 
for new ones? The next step when having reviewed this, is to update your security requirements so you know what to implement.

## We need requirements, not threats!
For our QuickRich app's authentication process our little brainstorming revealed a number of threats. Now is the time to plan 
how to make those problems go away. 

### Threats found from user story perspective

| Threat                | How to fix it             | Priority      |
|-----------------------|---------------------------|---------------|
|Use leaked passwords  |Create a password policy that checks if user passwords exist in leaked password lists. |High (easy to do) |
|Brute-force attack on passwords |Introduce lock-out after 10 failed attempts |High (easy to do + effective) |
|Phishing with fake login page |Add UI elements to remind the user to check URL before logging in. |Medium (easy to do, may help a little) |
|Phishing... |Send email alert when there is a login from unusual location with link to reset password. |Medium (requires more work, but can be effective in stopping phishing attacks) |
|SMS SIM swap |Use a 2FA app like Google Authenticator instead | High (easy to do, big improvement) |

All the requirements in the table above were found just form the user story - no data flow diagram needed. On the data flow diagram level we 
identified a few more, such as potential XSS and SQL injection. Those can be readily handled by good coding practice and the use of 
static analysis tools. What we have on top of this that is worth thinking about is API and database authentication and data transfer
security. Let's add those to our list of problems to fix. 

### Threats found from data flow diagram perspective

| Threat                | How to fix it             | Priority      |
|-----------------------|---------------------------|---------------|
| Hacker steals or brute-forces database connection |Only allow connections from the API through IP whitelisting or similar. <br> Use a strong password to avoid guessing/brute-forcing. <br> Make sure the database connection is encrypted. |High (easy to do, very effective) |
| Monkey-in-the-middle |HTTPS on all traffic over the internet | High (easy to do, dangerous to forget) |

## Test instead of testify
Now we have managed to find a number of threats to our authentication process that could potentially make that 
*worst thing that could happen* actually happen. We have also figured out ways to avoid this from happening. 
That has _zero value_ unless we actually implement this and know that it is working. Failing to do so could put us 
in a position where we have to respond to difficult questions from lawyers. Building test cases based on our threat model 
sounds more pleasant than being accused of breaching data protection laws in a trial - so let's build test cases!


| Security requirement                | Test case                   | Test method |
|-----------------------|-------------------------------------------|-------------|
| All passwords should be checked against a list of known leaked passwords. |Assert True: Setting a password that is contained in *blocklist-of-chopice*  returns FORBIDDEN. |Unit|
| Send email alert on suspicious logins. |Assert True: Warning email is sent to user when there is a login from an location not recently observed for that user |Unit|
| Add UI elements to remind users to check URL before entering username and password |Assert True: HTML element with textContent "Please verify that the URl of this page ends with 'richquick.io* before logging in." |Unit (preferred) or Manual (not ideal, needs to be written down somewhere).|


