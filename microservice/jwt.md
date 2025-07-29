---
title: JWT (JSON Web Token)
---

## Introduction

JWT is used to transmit verified information (claims) between parties. When
signed, it ensures data integrity—so if anyone tampers with the content, the
signature fails.

## JWT Structure

```
<encoded header>.<encoded payload>.<encoded signature>
```

### Header

A JSON object containing metadata about the token.

- `typ`: `JWT`
- `alg`: the algorithm used to sign the token
  - `HS256`: symmetric
  - `RS256`: asymmetric

### Payload

The token’s data, known as _claims_, expressed as JSON.

#### Claim types

- _Registered_: Standardized fields like `iss` (issuer), `sub` (subject), `aud`
  (audience), `exp` (expiration), `iat` (issued-at), `nbf`
- _Public_: Application or community-defined claims with names registered to
  avoid collisions.
- _Private_: Custom claims agreed upon by sender and receiver (e.g., `role`,
  `isAdmin`)

### Signature

Ensures _integrity_ (data hasn't been altered) and _authenticity_ (it comes from
the trusted source).

```
signature = HMAC‑SHA256(
  base64Url(header) + "." + base64Url(payload),
  secret_key
)
```

## JWE

**JWE (JSON Web Encryption)** ensures **confidentiality**, encrypting the
payload so that only intended recipients can read it, using symmetric or
asymmetric encryption.

## JWS

**JWS (JSON Web Signature)** provides **data integrity and authenticity**, using
a signature generated with a secret (HMAC) or private key (asymmetric) to ensure
the payload hasn’t been tampered with.
