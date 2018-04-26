---
layout: post
title: Code Reading - JWT
---

A code reading by [@spencer1248](https://github.com/spencer1248)

## JSON Web Tokens

Website: [jwt.io](jwt.io) (maintained by private company AuthO)

### JWT Basics

#### Base64 endcoding

JWTs use base64 encoding.

Why?

- Normalizes data into set of known characters
- No collisions with heading encodings
- Doesn't break JSON parsers
- Signature is binary
- Take binary data and convert into this common character set

Base64 basics

- 1 character, 64 different permutations
- 6 bytes with 4 characters
- When encrypting, keep in mind going from 6 bytes to 8 bytes increases size of data by 33%


#### Structure

Start with Base64 string

3 parts:

- header
- payload
- signature

`<header>.<payload>.<signature>`

Example - period delimited JWT with Base64:

`{"typ": "JWT", "alg": "HS256"}.{"aud": "learn", foo": "bar"}.<signature>` (signature is a binary string)

- Signed with key and algorithm (ex. HS256)
- Public and private claims
- Private claims are registered (ex. expiry)

#### General Concerns

- Don't use JWTs for sensitive information
- Don't use JWTs as a session replacement [[post](http://cryto.net/~joepie91/blog/2016/06/13/stop-using-jwt-for-sessions/)]


### 2Learn and JWTs

There are public JWT Devise strategies.

We didn't use any of those because:  
  - Too much custom stuff in our codebase and 2U's codebase
  - Will allow us to be more flexible in response to 2U's requests / requirements

Our custom JWT strategy (`JWTAuth`):  
  - integrated into our Rack Middleware
  - flip on using env variables
  - lives in front of Warden (may get moved further down the chain)

If you wanna play around with it, we have an example `.jwt_auth_inject.yaml` file you can use to test settings.
