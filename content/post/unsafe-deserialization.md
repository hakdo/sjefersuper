---
title: "Unsafe deserialization"
date: 2021-09-20
draft: false
tags: [vulnerabilities, owasp-top-10]
---
When we need to store complex data structures in a file or transfer
them over the network, we need serialization. Serialization is the process of 
converting the object into a flat format that can be transferred as a data sequence, 
such as a string. In most cases this is unproblematic if we are talking about 
static data. Sometimes we will need to serialize more dynamic data, such as 
classes, class instances or functions. Because of this, the reverse process of 
serialization - building up the original object again, needs to support building up 
classes, class instances and functions again. This is only possible with some evaluation 
of the information in the serialized data. Many libraries do this unserialization process in 
an unsafe manner, using `eval` based routines. If an attacker controls the data that will be 
deserialized, the attacker can then achieve bypass of application level controls on data assignment, 
as well as remote code execution. 

## Defending against unsafe deserialization
We need to defend against attacks likt this. The best way is to avoid deserializing data at all, if we can. 
If we cannot do that, we should be very careful about the integrity of the data we deserialize. There are also 
some libraries that are safer to use than others. 

1. Don't deserialize unless you must, and never deserialize user controlled data.
2. Verify the integrity of the data when you need to deserialize objects.

There is a good write-up on this topic from Acunetix: [Deserialization vulnerabilities: attacking deserialization in JS](https://www.acunetix.com/blog/web-security-zone/deserialization-vulnerabilities-attacking-deserialization-in-js/).

## Python's pickle module
A simple example of unsafe deserialization can be found in Python's pickle module. 
This is a very convenient module to use as it can serialize almost any valid Python
object. Unfortunately, it is insecure and can easily be used by an attacker to achieve
arbitrary code execution. The [documentation states](https://docs.python.org/3/library/pickle.html): 

> Warning The pickle module is not secure. Only unpickle data you trust.

Consider an application where we load some user data from a file: 

```python
import pickle

with open("backup.data", "rb") as file:
    pickle_data = file.read()
    mydata = pickle.loads(pickle_data)
```

This would be completely fine, if only we could guarantee that the file "backup.data" has not been 
tampered with. What we intend for the data to look like is perhaps something like this: 

```python
profile = {
    'username': 'donkey',
    'status': 'golden'
}
```

Let's say an attacker comes along and has write access to the `backup.data` file. The attacker then overwrites the file 
with a pickled version of the following class:

```python
class EvilPickle():
    def __reduce__(self):
        import os
        return(os.system, ('<evil-system-level-code-here>')
```
This would give the attacker remote code execution and direct access to the underlying operating system. 
The reason is that `__reduce__`is a *magic function* that will be executed directly during deserialization
(unpickling in Python terminology). 

For a good write-up on pickle deserialization exploitation at machine instruction level, see [DANGEROUS PICKLES — MALICIOUS PYTHON SERIALIZATION](https://intoli.com/blog/dangerous-pickles/).

## What about NodeJS?
We can do exactly the same "demo" with NodeJS, if you rely on one of several exploitable serialization libraries. 
One of the most popular (and easily exploitable) is [node-serialize](https://www.npmjs.com/package/node-serialize).
The NPM page doesn't really tell you anything about the dangers about using its deserializer but "npm audit" will 
tell you that you are moving into the danger zone. 

```
node-serialize  *
Severity: critical
Code Execution through IIFE - https://npmjs.com/advisories/311
No fix available
```
Yet, this library has more than 1400 weekly downloads.

Let's say we now reimplemented our Python application in Node. Here's the code that 
loads the `backup.data` saved to disk: 

```javascript
const serializer = require('node-serialize')
const fs = require('fs');

fs.readFile("backup.data", "utf-8", function (err, data) {
    if (err) {
        console.log("Error reading profile data: ", err)
    } else {
        // unserialize profile data
        var profile_data = serializer.unserialize(data)
        console.log(profile_data)
    }
});
```

The data we are intending to load still has the same structure as in the Python case. 
But also now, an evil hacker has managed to overwrite our file with their own payload.

```
{"username":"donkey","status":"golden","rce":"_$$ND_FUNC$$_function(){\n    require('child_process').exec('say \"Dangerous hackers are taking over!\" && open https://youtu.be/V4MF2s6MLxY', function(error, stdout, stderr) { console.log(stdout) });\n  }()"}
```

If you deserialize the string above using node-serialize's "deserialize" function on a Mac, the computer will 
speak to you and open an enlightening [YouTube video](https://youtu.be/V4MF2s6MLxY) in your browser for you. 

Here's how this looks in practice:
{{< youtube ffoQ0CPKh8Y >}}

This attack uses an immediately invoked JavaScript function in the payload, and because the deserializer relies on `eval`, 
this function is executed upon deserializaion.
