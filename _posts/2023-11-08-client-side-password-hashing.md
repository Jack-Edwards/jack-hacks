---
layout: post
title: Taking a look at hashing passwords on the client
description: What the title says
date: 2023-11-08
---

Basic authentication in the modern world is pretty well understood.

1. The client application accepts an email address and password from the user.
2. The client applications transmits those credentials to the server, over an encrypted connection.
3. The server hashes the provided password and compares the output to the hash saved on the server.

What could go wrong?

tl;dr: The user uses the same password across multiple services and your system is not as closed as you think.

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

I have experienced this several times.  The first time we caught it in QA.  The second time I opened up a legacy system and observed the issue had already existed for years.

### Open Databases

Now consider the database server. One of the most isolated and secure servers in the entire organization. The data held on this server is your organization's most precious resource.

This is the holy grail for every adversary attempting to breach your network. Adversaries the size of nation-states are after this information. The majority of us are foolish to think we could prevent such a breach.

As much as we all want to deny it, our databases are often exposed in ways we don't even know. Or they will become exposed over time as conditions change and we fail to keep up. How does the saying go... "defenders must protect against every scenario; attackers only need to exploit a single flaw." Overly simplistic, but you get the point.

That being said, I strongly believe a responsible authentication provider should takes measures to *mitigate* the damage caused by a database breach. Not to protect the organization's interests (at least not directly), but to protect their users from the fallout.

Those words sounds nice, but what am I talking about?

Most server-side password hashing algorithms are designed for a specific use-case: the server. The algorithm needs to stress server in ways that will not exhaust the server's resources, preventing it from processing other requests at the same time. PBKDF2, the standard password hashing algorithm used in ASP.NET, is designed to stress the CPU. The memory and disk are left alone. This means an adversary with access to hashed passwords, hashed using PBKDF2, only need to design a system with the CPU in mind.

### Enter the client-side pre-hash

There are two significant benefits to performing a "pre-hash" on the client. Whether your applications or business will allow for a client-side pre-hash to be performed is another matter. For now, let's assume you would be able to implement something like this.

The first benefit to performing a round of password hashing on the client is to make brute-forcing leaked passwords even more difficult by requiring the client to perform more rounds of the same password hashing algorithm or by introducing a second password hashing algorithm.

The second benefit to performing client-side pre-hashes is the user no longer needs to trust your servers with their actual, plaintext passwords. Your servers only see fixed-length password hashes. This is a win-win situation for your users and your business.

### Pit falls

A couple things to keep in mind when implementing a client-side pre-hash.

First, you must still perform sufficient server-side password hashing. Though your servers are no longer aware of what your users plaintext passwords are, there is still the matter of making `Login` attempts sufficiently difficult for adversaries attempting to gain access to user accounts. If all an adversary wants to do is gain access to a user account, then they wouldn't need to bother with replicating the client-side pre-hash system. All the adversary needs to do is replace their plaintext password guesses with random guesses that adhere to the pre-hash format (length, patterns, etc).

And the second thing to remember is that all clients must agree to and implement the same password hashing algorithm and parameters. Otherwise you may have users who signed up on your web client unable to login on your Android or iOS applications, for example.

### How much hashing on the client vs the server

The client-side pre-hash is designed to obfuscate and protect the user's plaintext password from your own business, as well as adversaries who may have gained access to your systems. You should perform your own risk analysis to determine how much pre-hashing is warranted against how long your users are willing to wait for the client-side pre-hash to run.

The server-side password hash, in this case, is only designed to obfuscate and protect the output of the pre-hash. While the user's plaintext password should now be sufficiently protected, you still want to prevent adversaries from gaining access to the user's account. Again, you must perform your own risk analysis to determine how much server-side hashing is warranted.

While password hashing on both the client and server would be ideal, you should never perform any _less_ password hashing because you think the other system will make up the difference. Again, the pre-hash is intended to protect the user's secret plaintext password while the server hash is intended to protect against unauthorized entry to the system.

### Recommendations

 If you are interested in implementing a client-side pre-hash in your application, I recommend using Libsodium. Any other cross-platform cryptography library would work fine, but it really helps to have the same library available across platforms.

Store all client-side pre-hash parameters with the output. Send those parameters to the server along with the output of the pre-hash. These parameters are not secret. You will need these parameters handy if you ever intend to strengthen the pre-hash later on and need to perform a password migration.

Choose the right algorithm for the hardware. PBDKF2 is a decent password hashing algorithm to run on the server. It relies primarily on CPU and will not exhaust your server, unlike memory or GPU intensive algorithms. On the other hand, Argon2id is intended to consume way more memory and is ideal for client-side pre-hashes, but would quickly exhaust your server's memory in case your server ever needs to hash multiple passwords concurrently.
