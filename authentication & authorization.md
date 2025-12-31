Authentication and Authorization
=================================================================

This topic answers two very basic questions for any backend system:

1.  **Who are you?**   
2.  **What are you allowed to do?**
    
Everything else builds on these two questions.

1\. Authentication vs Authorization
---------------------------------------------------------------

### Authentication (Who are you?)

Authentication is about **identity**.

Examples:
*   Login with username and password
*   Login with Google
*   Login with fingerprint

Question it answers:

> “Are you really who you claim to be?”

### Authorization (What can you do?)

Authorization is about **permissions**.

Examples:
*   Can you view this page?
*   Can you delete this record?
*   Are you an admin or a normal user?
    
Question it answers:

> “Now that we know who you are, what are you allowed to do?”

### Simple Rule to Remember
*   **Authentication comes first**
*   **Authorization comes after**

You cannot authorize someone you have not authenticated.

2\. How Authentication Started (Very Simple History)
----------------------------------------------------

Authentication did not start with computers.

### Early Human Society (Trust-Based)

In small villages:
*   Everyone knew everyone
*   A trusted elder could vouch for you

Authentication was:
*   Based on **human trust**
*   Worked only in small groups
    

Problem:
*   Does not scale
    

### Seals, Tokens, and Possession

Later societies used:
*   Wax seals
*   Rings
*   Physical marks

Authentication became:
*   “Something you have”
    
Problem:
*   Can be stolen or forged

### Passwords and Shared Secrets

With telegraphs and early communication:
*   People agreed on secret phrases
*   Only trusted parties knew them

Authentication became:
*   “Something you know”
    
Problem:
*   Can be guessed or leaked

### Computers and Password Storage

Early computers:
*   Stored passwords in plain text
*   Anyone could read them

This caused security incidents.

Solution:
*   Hash passwords
*   Never store raw passwords

This is still true today.

### Modern Cryptography

Breakthroughs like:
*   Public-key cryptography
*   Secure key exchange

Enabled:
*   Secure login over the internet
*   Encrypted communication

This is the foundation of modern auth systems.

### Internet Scale Problems

As the internet grew:
*   Millions of users
*   Attacks became common

Result:
*   Passwords alone were not enough

Solution:
*   MFA (Multi-Factor Authentication)

### Modern Era (What We Use Today)

Modern systems use:
*   OAuth
*   JWT
*   Passwordless login
*   Zero-trust security
*   External auth providers

The goal:
*   Secure
*   Scalable
*   User-friendly

3\. Why Sessions Exist
----------------------

### The Problem

HTTP is **stateless**.

That means:
*   Server forgets everything after each request
    
But real apps need:
*   Logged-in users
*   Shopping carts
*   Dashboards

### What Is a Session?
A **session** is how the server remembers you.

Simple flow:
1.  You log in
2.  Server creates a session ID
3.  Session ID is stored on server
4.  Session ID is sent to browser (cookie)
5.  Browser sends it back on every request

Now the server knows:
*   This request belongs to this user

### Where Sessions Are Stored
Over time, storage evolved:
*   Files
*   Databases
*   In-memory stores (Redis)

Sessions are **stateful**:
*   Server stores data

4\. JWT (JSON Web Token) – Why It Exists
----------------------------------------

### The Problem with Sessions at Scale

In large systems:
*   Many servers
*   Many regions
*   Sessions need syncing

This becomes:
*   Expensive
*   Complex

### What Is a JWT?

A JWT is a **self-contained token**.

It contains:
*   User ID
*   Role
*   Expiry time
*   Signature

The server does **not** store it.

### How JWT Works (Simple)

1.  User logs in
2.  Server creates a JWT
3.  Server signs it
4.  Client stores it
5.  Client sends it with every request
6.  Server verifies signature

No database lookup needed.

### Pros of JWT

*   Stateless
*   Scales well
*   Great for microservices
*   Easy to share across services

### Cons of JWT
*   Hard to revoke
*   Valid until expiry
*   Logout is not instant

Once issued, it lives until it expires.

### Hybrid Approach

Some systems:
*   Use JWT
*   Maintain a blacklist for revoked tokens

This fixes revocation but adds state again.

5\. Cookies (How Tokens Are Sent Automatically)
-----------------------------------------------

Cookies are:
*   Small data stored in the browser
*   Automatically sent with requests

Why they matter:
*   Browser handles auth automatically
*   No manual header management

Cookies are often used to store:
*   Session IDs
*   JWTs

Browsers also enforce:
*   Domain isolation
*   Security flags

6\. Types of Authentication (Simple Breakdown)
----------------------------------------------

### Stateful Authentication (Sessions)

How it works:
*   Server stores session data
*   Client sends session ID

Pros:
*   Easy logout
*   Easy revocation
*   Clear active sessions

Cons:
*   Harder to scale
*   Needs shared storage

Best for:
*   Traditional web apps

### Stateless Authentication (JWT)

How it works:
*   Server issues token
*   No server-side storage

Pros:
*   Scales very well
*   Great for APIs
*   Microservice friendly

Cons:
*   Token revocation is hard

Best for:
*   APIs
*   Mobile apps
*   Distributed systems

### API Key Authentication

Used when:
*   Machines talk to machines
*   No human UI involved

How it works:
*   Generate a long random key
*   Send it with each request

Pros:
*   Simple
*   Easy to rotate

Cons:
*   No user context
*   If leaked, full access

Best for:
*   Developer APIs
*   Internal services

7\. Practical Advice from the Video
-----------------------------------

For real systems:
*   Auth is hard to get right
*   Mistakes are expensive

Recommendation:
*   Use trusted providers (Auth0, Clerk, etc.)
*   Avoid building everything yourself in production

Final Simple Summary
--------------------

*   **Authentication**: who you are
*   **Authorization**: what you can do
*   Sessions store state on the server
*   JWTs store state in the token
*   Cookies automate sending auth data
*   Stateful = control, Stateless = scale
*   API keys are for machines, not users

### One-line takeaway

> **Authentication proves identity, authorization enforces permissions, and modern systems choose the mechanism based on scale and security needs.**
