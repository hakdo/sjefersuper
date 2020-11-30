---
title: "Idor"
date: 2020-11-30T18:55:17+01:00
draft: false
tags: [vulnerabilities, owasp-top-10]
---
Insecure direct object references are very common, and is often referred to as "IDOR". 
This occurs when a user input is directly used to access a data object, without 
appropriate authorization checks. A common example is a URL with an ID number as a parameter, 
where users can access data meant for other users simply by changing that number in the URL. 

The implications of this depends on the accessible endpoints. The most common case is that a GET 
request to and endpoint has this type of vulnerability, allowing confidentiality to be broken. 

This is an example of [broken access control](https://owasp.org/www-project-top-ten/2017/A5_2017-Broken_Access_Control).

## Practices to avoid IDOR
The most obvious fix is not to expose direct object references at all in the user interface. 
For example, if your customer ID is a primary key in your database, instead of using a url like this: 
```
https://example.com/profile/customer/1234
```
you could use a secondary field to store a user ID that cannot be easily guessed or identified by 
following a number sequence, for example a UUID or a random string. 

In addition to that, you should add authorization checks in your backend, to see if the object the 
application is trying to access should be accessible by the user making the request. 

Consider a Django application using direct object references (very common in this framework). 
Perhaps the user is trying to find data on their own diary in an online diary application. 
Each diary has an entry in a *Diaries* table in the database. It is then very tempting to make a request 
like 

```python
user_diary = get_object_or_404(Diaries, pk=27)
```

where 27 is a parameter coming from the URL. Without any further logic this would be an example of an
IDOR. Let's say the table also has an *owner* field as a foreign key to the "User" table in the 
application. An easy fix would then be to use 

```python
user_diary = get_object_or_404(Diaries, pk=27, owner=request.user)
```
where the application would return a 404 Not Found status if the requested object is not owned by 
the currently logged in user. 

You could of course combine these techniques by giving each diary entry a "diary_id" field with a 
random token, so that a request could look like this: 

```python
try:
    user_diary = Diaries.objects.get(diary_id=salkfwofalskjwqoidsoij, owner=request.user)
    context = {
        'diary': user_diary,
        ...
    }
    return render(request, 'mytemps/diary.html', context)
except Exception as e:
    logger.info('Diary not found for user request: ', e)
    return 404Response
```
To summarize: 
- Always check authorizations on user requests for data
- Consider if you should mask the object references in the frontend (usually a good idea)