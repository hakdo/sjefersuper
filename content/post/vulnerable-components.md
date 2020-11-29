---
title: "Vulnerable Components"
date: 2020-11-29T19:02:29+01:00
draft: false
tags: [vulnerabilities, owasp-top-10]
---
When we build software, we rely third-party libraries. If those libraries have 
vulnerabilities, that can quickly lead to exploitable vulnerabilities in our application.
Using 
[components with known vulnerabilities](https://owasp.org/www-project-top-ten/2017/A9_2017-Using_Components_with_Known_Vulnerabilities) 
is number 9 on the OWASP Top 10 list.

{{<figure 
    alt="Library with lots of books"
    src="/images/library.jpg"
    caption="Having a lot of dependencies in your code can greatly expand your attack surface." >}}

There at two types of vulnerabilities we should consider in third-party code: 

- Known vulnerabilities that have been found by researchers and reported to the project
- Malicious code in libraries, where someone on purpose has injected evil code into a library

Protecting against the first type of vulnerability can be done automatically using tools that 
compare the libraries in your project with a database of known vulnerabilities and will alert you, 
or even automatically fix it for you. There are both open source and commercial closed-source 
variants of such tools. Among open-source tools,
OWASP [dependency check](https://owasp.org/www-project-dependency-check/) is perhaps the most
commonly used. 

## Malicious libraries
Protecting against malicious libraries is much harder. Malicious libraries typically exist in 
two varieties: 

- Pure evil libraries using typosquatting (using a name that is similar to a well-known library)
and fake reviews to drive implementation
- Compromised legitimate libraries where attackers are injecting evil code an otherwise useful library

Both are hard to detect for the developer. Fake libraries trying to impersonate projects that are 
popular tend to have short life-spans. Checking the version or commit history of a library can be 
an indicator of whether this is the real thing or not. 

Injected evil code into legitimate libraries can be even more difficult to discover. A thorough code 
review could help, but if your project depends on 12000 third-party libraries that is not 
very realistic. 

## Is the vulnerability reachable in the code?
Whether your application becomes vulnerable by using a library with known vulnerabilities, depends 
on whether the vulnerable code can be triggered by an attacker with access to the application 
in its runtime environment. Some commercial dependency management tools try to evaluate whether a 
vulnerable is "reachable" or "effective". This can be useful for prioritizing fixes but in principle 
you should aim to now have references to libraries with known vulnerabilities in your codebase. 

## Protective measures to reduce the attack surface through third-party libraries
The following practices can help reduce the third-party code attack surface, and thus your 
project's risk level. 

1. Check dependencies for known vulnerabilities using a tool in your pipeline
2. When using a third-party library with a low number of users, check if it is frequently updated, 
if the library has a commit or release history that makes sense
3. If you can, prioritize simplicity by flattening your dependency tree. If your project has 12.000
dependencies there is no way you can control your own attack surface. Many libraries offer functionality 
you can easily implement directly with a few lines of code; when this is the case, write it yourself. 

## A note on Github and open source vulnerabilities 
If you use Github for your code repositories, some security features helping you manage third-party 
libraries come with that package. 
[Dependabot](https://github.blog/2020-06-01-keep-all-your-packages-up-to-date-with-dependabot/)
will alert you about vulnerable dependencies and create a pull request to fix it. 

You can also use the "Dependency Graph" tool under "Insights" for your repository to see the 
dependencies your project depends on.

## Further reading elsewhere
[Darkreading: Unpatched Open Source Libraries Leave 71% of Apps Vulnerable](https://www.darkreading.com/application-security/unpatched-open-source-libraries-leave-71--of-apps-vulnerable-/d/d-id/1337856)

For a more academic discussion on detection of reachable vulnerabilities: 

Ponta, S.E., Plate, H. & Sabetta, A. [Detection, assessment and mitigation of vulnerabilities in open source dependencies](https://link.springer.com/article/10.1007/s10664-020-09830-x). Empir Software Eng 25, 3175–3215 (2020).
