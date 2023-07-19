---
layout: post
title: Taking a look at hashing passwords on the client
description: What the title says
date: 2023-07-20
---

Basic authentication in the modern world is pretty well understood.

1. The client application accepts an email address and password from the user.
2. The client applications transmits those credentials to the server, over an encrypted connection.
3. The server hashes the provided password and compares the output to the hash saved on the server.

What could go wrong?

### Leaky code

Put yourself in the position of the authentication server.

You just received a request that passed your CORS policy.
This request contains a valid email address and a password that meets or exceeds your arbitrary password requirements.
Nice.

What are you going to do with this request?

Ideally you do the bare minimum. Hash the password, compare the password to the stored hash, then return either an authentication token or HTTP 4XX.

Rarely is that the case though. Developers often add verbose logging to their systems. The logs might be written explicitly and only contain pieces of information. Or the logs might be written automatically and will write as much information possible to the log file, not knowing what information will be important during the next customer escalation. Request headers. Request bodies. Plaintext credentials. Anybody with access to these log files would have access to plaintext user passwords. This is concerning.

Now imagine your organization uses a third-party observability service with a graphical log explorer. Not only do members of your organization have access to the plaintext user passwords, but so does the third-party service. This too, is concerning.

Logging plaintext user credentials is usually done unintentionally, but with the best of intentions. The developer who added the `VerboseLoggingMiddleware` did so without any concern to the pre-existing `Login` or `SignUp` server endpoints. Those endpoints were probably added by one founding developers of the project, who may not even be with the organization anymore. Yet, 5 years later and in response to a string of escaped defects, management decided they needed better audit logs to pre-empt the next customer escalation. So the next developer in line gets the new middleware working, sees absolutely everything about every request get logged (as expected), and the change gets deployed to production.

I have experienced this several times first hand.  The first time we caught it in QA.  The second time I opened up a legacy system and observed the issue had already existed for years.

### Open Databases

Now consider the database server. One of the most isolated and secure servers in the entire organization. The data held on this server is your organization's most precious resource.

This is the holy grail for every adversary attempting to breach your network. Adversaries the size of nation-states are after this information. The majority of us are foolish to think we could prevent such a breach.

As much as we all want to deny it, our databases are often exposed in ways we don't even know. Or they will become exposed over time as conditions change and we fail to keep up. How does the saying go... "defenders must protect against every scenario; attackers only need to exploit a single flaw." Overly simplistic, but you get the point.

That being said, I strongly believe a responsible authentication provider should takes measures to *mitigate* the damage caused by a database breach. Not to protect the organization's interests (at least not directly), but to protect their users from the fallout.

### Enter the client-side pre-hash



damage mitigation
if pre-hashed, the server only knows the password for itself, not other services

### Pit falls

you must still perform sufficient server-side hashing

all clients must agree on the algorithm and parameters

### How much hashing on each side

The pre-hash is designed to obfuscate the plaintext password
- you must perform adequate rounds of password hashing, appropriate for this risk

the server hash is designed to obfuscate the authentication credential
 - you must perform adequate rounds of password hashing, appropriate for this risk

 In a combined system, you only need to perform the greater amount of password hashing for either risk

 ### Recommendations

 libsodium (any cross platform library)

 store all parameters with the password (both client and server side)

 choose algorithms suited for the platform (argon for client vs pbkdf for server)

