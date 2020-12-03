---
title: "Secrets in applications"
date: 2020-12-03T17:54:39+01:00
draft: false
tags: [secrets, owasp-top-10]
---
Proper handling of application secrets is important. Leaking secrets such as 
database credentials or user tokens is unfortunately quite easy to do. 

Applications are generally poor at keeping secrets. They tend to 
[leak them in logs](../secrets-in-logs), in stack traces and sometimes even 
in messages back to users. 

Another problem with many application secrets is that they are difficult to change. 
Because of this they tend to remain the same over a long time, perhaps even forever. 
The combination of long-lasting credentials and high likelihood of credentials leaks 
should be considered an unacceptable risk. 

## What we want
A good way to manage secrets should conform to the following requirements: 

1. They should be easy to rotate
2. They should have limited lifetime by default
3. A credential leak should have a limited blast radius. If you use the same database connection string 
in many applications, compromising the secret from one application would cause damage to all
applications.
4. They should not be exposed in plain text

## Using a safe secrets store
The best way to manage secrets in an application is to use a "secrets store". All the big cloud 
vendors have their own offerings, or you can install a secret store yourself. A popular choice is 
[Hashicorp Vault](https://www.vaultproject.io/) which comes both in an open-source variant and an 
"enterprise" version with more features for money. The point of using a secret engine is that it should 
provide: 

- Secure cryptographic operations
- Secure and practical interfaces for injecting secrets in applications
- Easy ways to rotate credentials
- Ability to create ephemeral credentials
- Access control helping to implement the principle of least privilege

## At least do this
If you are not using a secrets manager, here's the minimum solution: 

- No hard-coded secrets
- If you are using configuration files, make sure the permissions are set correctly
- Environment variables are better than config files as they can't be extracted simply through file 
permission errors. 
- Make sure you do not submit config files to source version control (git)

