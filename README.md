# README jwt-secure-communication

## Purpose
This is the Deceptively Simple Technologies (DST) JSON Web Token (JWT) Secure Communication Standard, which defines and utilizes open standards and a code library to **represent, sign, and transmit information securely** between two parties as JSON (JavaScript Object Notation) objects.

## Standard
**JSON Web Token (JWT)** [RFC 7519](https://www.rfc-editor.org/rfc/rfc7519) and JSON Web Signature (JWS) [RFC-7515](https://www.rfc-editor.org/rfc/rfc7515) are open standards, used to represent, sign, and transmit information securely between two parties as JSON (JavaScript Object Notation) objects. A number of third-party code libraries are available to generate, decode, and verify JWT for use with various computer programming languages.

We have **standardized on a single library** (please see below) for use in the DST Adaptív Application Foundation (AAF)™ and other DST systems, in order to reduce complexity and possible errors. Note: DST AAF is based on the Linux-compatible Microsoft .NET Core platform. Note: Engineers should be aware that **other third-party libraries may** implement the JWT standard in ways that **introduce security vulnerabilities**. We have exercised caution to select a secure library and to use JWTs as documented and intended and in ways that minimize the likelihood of these vulnerabilities. The approach outlined here is intended to recommend such an approach.

Note: Our recommended JWT library is **[JsonWebToken]**(https://github.com/ycrumeyrolle/jwt)

* It supports all standard JWT features except EdDSA signature algorithm (EdDSA is unsupported by most if not all of these libraries)
* Very high performance on .NET Core due to its use of modern APIs
* No dependency on NewtonSoft JSON
* Signature verification allows specifying the algorithm rather than just pulling it from the header (this is a big plus for security)
* Clear and understandable API. On par with the JWT library in that respect.

Note: There is an older package called JsonWebTokens (plural), which should NOT be used. The version number of this older package is higher than the one we’re using, and it was released quite some time ago. It supports fewer frameworks and is a completely different and unrelated library to the one we've recommended.

We will use **shared secrets** generated with the HMAC-SHA256 algorithm to digitally sign our JWTs, rather than an asymmetric public/private key pair generated using RSA or ECDSA algorithms due to related weaknesses in many several third-party libraries. Our JWTs will consist of Bas e64Url-encoded (to remove spaces and problematic punctuation) header and payload and the 256-bit shared secret signature sections (please see the examples below), separated by the dot (.) character, i.e. HEADER_DATA.PAYLOAD_DATA.SIGNATURE_DATA These signed JWTs can verify the integrity of the information they contain because the signature is calculated using the header and the payload data as inputs.

Our JWT **header** sections will contain the "alg" (cryptographic algorithm used in the signature) property (defined in RFC 7519), the "**typ**" property, and the "**kid**" (key id) header parameter (defined in RFC7515) to facilitate coordinating shared secret (key) changes (please see the example below).

Our JWT payload sections will contain the "**iss**" (Issuer), "**sub**" (Subject), "**aud**" (Audience), "**exp**" (Expiration Time in number of seconds from 1970-01-01T00:00:00Z UTC until the specified UTC date/time, ignoring leap seconds or "Seconds Since the Epoch, defined in IEEE Std 1003.1 2013 Edition [POSIX.1]), "**iat**" (Issued At also in in number of seconds from 1970-01-01T00:00:00Z UTC, etc), "**nbf**" (Not Before also in in number of seconds from 1970-01-01T00:00:00Z UTC, etc), and "**jti**" (JWT UUID) **registered claims** – predefined claims that are not mandatory but recommended (please see the example below).

This results in a 326-character JWT: ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IkRzdFBsYXRmb3JtLTAwMS1ERVYifQ.eyJpc3MiOiJEc3RDbGllbnQiLCJzdWIiOiJMb2dJbiIsImF1ZCI6IkRzdFBsYXRmb3JtIiwiZXhwIjoiMTY0MDk5NTMyMCIsImlhdCI6IjE2NDA5OTUyMDAiLCJuYmYiOiIxNjQwOTk1MjAwIiwianRpIjoiODU2YTNjMTAtOGM1NC00ZjBkLTllMzAtZWVjOTUzZmFmYmYzIn0.N0OKTlktU_4ejvTGEu1xtWpxjmKmYdJM0faHQIn8Oks```

Our JWT **payload** sections will also contain a "**body**" **private claim**, containing the business process request parameters, which will be passed to the workflow model and decision engine (please see the example below).

This results in a 457-character JWT: ```eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IkRzdFBsYXRmb3JtLTAwMS1ERVYifQ.eyJpc3MiOiJEc3RDbGllbnQiLCJzdWIiOiJMb2dJbiIsImF1ZCI6IkRzdFBsYXRmb3JtIiwiZXhwIjoiMTY0MDk5NTMyMCIsImlhdCI6IjE2NDA5OTUyMDAiLCJuYmYiOiIxNjQwOTk1MjAwIiwianRpIjoiODU2YTNjMTAtOGM1NC00ZjBkLTllMzAtZWVjOTUzZmFmYmYzIiwiYm9keSI6eyJVc2VyRW1haWxBZGRyZXNzIjoiYW5keS5hbmRlcnNvbkBleGFtcGxlLmNvbSIsIlVzZXJQYXNzd29yZEhhc2giOiJ2M3Q1dnNMTXJ0ZE16c2lsZ1d5cCJ9fQ.Nbu0IsDAjnNw8xZMA9GPudpKhdEEgB5EwTSBYkNatas```

While JWTs can be **encrypted** (rather than just signed) using the shared secret key, encrypting the JWT may not be necessary unless it contains sensitive information because the **HTTP communication channel** over which requests and responses will be sent **should always be secured with** Transport Layer Security (**TLS**) cryptographic protocol. Note: This means that all communication between systems must be secured with TLS.

Each system (DstClient, DstPlatform) must **validate** the JWTs it receives. This includes **rejecting JWT with missing signature**, validating the **JWT signature contents**, **rejecting expired JWT** and **JWT with lifetimes greater 2 minutes**, **rejecting duplicate JWT** (based on the jti claim), validating the **JWT payload contents**. DstPlatform may also validate JWT size (< 1,000 characters) due to workflow engine restrictions on data element length, but this should not be a factor with the limited number of parameters DstClient will be sending in each request.
      
Our JWT will be sent in the **body** of an **HTTP POST** rather than in an Authorization: ```Bearer <token>``` HTTP Header record primarily because parsing the contents of request bodies requires less .NET Core custom middleware.

**DstPlatform will respond** to valid JWT from DstClient **with its own JWT**, **signed** with the **same shared secret**, containing a payload with a **"body" private claim**, e.g.

```{
  "iss": "DstPlatform",
  "sub": "LogIn",
  "aud": "DstClient",
  "exp": "1640995320",
  "iat": "1640995321",
  "nbf": "1640995201",
  "jti": "1406f626-aa3c-4f66-8c7e-419611efd1e2",
  "body": {
    "UserAuthToken": "seyQeJlAWE5EBX3UL1d7",
    "UserRefreshToken": "yhoWtYxqnw0fc5Oi6Gor"
  }
}```

## Setup
1. Install the 

## Use
lksjaflsjdf

## Legal
Copyright © 2022 Deceptively Simple Technologies Inc.  All rights reserved.
