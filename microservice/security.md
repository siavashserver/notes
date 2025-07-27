---
title: Security
---

## Authentication methods

| Method         | Use Case                              | Transport        | Stateful? | Strengths                          | Weaknesses                                    |
| -------------- | ------------------------------------- | ---------------- | --------- | ---------------------------------- | --------------------------------------------- |
| Basic Auth     | Internal tools, quick APIs            | Headers (base64) | No        | Easy to set up                     | Insecure unless HTTPS, sends creds every time |
| Cookie-Based   | Traditional web apps                  | Cookies          | Yes       | Built-in with .NET/ASP.NET         | CSRF risk, limited SPA support                |
| JWT            | SPAs, mobile, APIs                    | Headers (Bearer) | No        | Stateless, scalable, cross-domain  | Token size, revocation tricky                 |
| OAuth2         | 3rd-party access (Google, Facebook)   | Redirect + token | Mixed     | Delegated access, secure           | Complex flows                                 |
| OpenID Connect | Identity + Profile info (Login via X) | Redirect + token | Mixed     | Identity layer on top of OAuth2    | Complex to set up                             |
| SAML           | Enterprise SSO (old but still common) | XML + Redirects  | Yes       | Federated auth, widely used in SSO | Verbose, XML-based                            |
| OTP            | MFA (TOTP, Email, SMS)                | User input       | Mixed     | MFA support, secure                | User friction                                 |

### Basic authentication

The client sends a Base64-encoded `username:password` in an `Authorization`
header on every request.

```
Authorization: Basic base64(username:password)
```

### Cookie based

The server issues an encrypted session cookie after login; the browser sends it
on every subsequent request.

- Client sends login information with form fields
- Server validates and issues cookie
- Browser stores cookie
- Making requests to server automatically includes cookie
- Server middleware validates cookie -> authenticated user

### SAML

Security Assertion Markup Language (SAML) uses XML assertions for SSO in
enterprise apps.

- User tries `/app` -> SP (Service Provider) sends SAML AuthnRequest to IdP
  (Identity Provider)
- IdP authenticates user -> returns SAMLResponse (base64 XML)
- SP validates signature, creates session

### One-Time Password (OTP)

OTP adds time-based or counter-based codes (TOTP/HOTP) to your login process.
Google Authenticator is a common TOTP client.

- Server generates secret and shares via QR code
- User scans QR; app generates 6-digit code every 30 seconds
- User submits code; server verifies with same secret

### Multi-Factor Authentication (MFA)

MFA combines two or more factors (password + OTP, hardware key, etc.) to harden
security.

- Password verification
- OTP, hardware token, or biometric check
- On success, issue session or tokens

### JWT with refresh token

- Client POSTs credentials -> server returns `{ accessToken, refreshToken }`
- Client calls API with `Authorization: Bearer accessToken`
- On HTTP 401, client calls `POST /token/refresh` with `refreshToken`
- Server validates and issues a new pair

### OAuth2

OAuth 2.0 is an authorization framework that enables third-party applications to
obtain limited access to a user's resources without exposing credentials. **It's
not an authentication protocol** (though it's commonly used with OpenID Connect
for that).

| Term                     | Description                                      |
| ------------------------ | ------------------------------------------------ |
| **Resource Owner**       | The user who authorizes access to their account. |
| **Client**               | The application requesting access.               |
| **Authorization Server** | Issues tokens after authenticating the user.     |
| **Resource Server**      | Hosts the protected resources (APIs).            |
| **Access Token**         | Short-lived credential to access resources.      |
| **Refresh Token**        | Long-lived token to obtain new access tokens.    |

#### OAuth2 Authorization Flows (Grant types)

- Authorization Code Flow
  - App redirects to the Authorization Server login screen.
    - `response_type`: specifies authorization flow
      - `code`: Client wants an authorization code (**recommended, use with
        PKCE**)
      - `token`: Client wants an access token directly
      - `id_token`: (OIDC only) Client wants ID Token only
      - `id_token token`: (OIDC only) Return ID Token + Access Token
      - `code id_token`: (OIDC only) Return code + ID Token
    - `client_id`: identifier of the client
    - `scope`: scope of the requested token
    - `state`: value set to maintain state between request and callback
      (optional)
    - `redirect_uri`: client callback URI for the authorization response
      (optional)

    ```
    https://c2id.com/login?
    response_type=code
    &client_id=s6BhdRkqt3
    &scope=myapi-read%20myapi-write
    &state=af0ifjsldkj
    &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcallback
    ```

  - After login, the server redirects back with an **authorization code**.
  - Frontend (with PKCE) sends the code to the backend (See dedicated PKCE
    section).
  - Backend exchanges the code for an **access token** and optionally a
    **refresh token**.
    ```json
    {
      "access_token": "XZPURHNPWIDF",
      "token_type": "Bearer",
      "expires_in": 3600,
      "scope": "myapi-read myapi-write"
    }
    ```

- Client Credentials Flow

  - Best for machine-to-machine (M2M) or service-to-service.
  - The client (.NET service) authenticates itself via `client_id` and
    `client_secret`.
  - Receives access token to call APIs (no user context).

- Password Grant Flow (Deprecated)

  - Only used in trusted applications (NOT recommended anymore).
  - Allows clients to **collect the user's username and password directly**, and
    send them to the authorization server to obtain tokens.

- Refresh Token: An OAuth 2.0 credential issued alongside an access token that
  enables the client to obtain a new access token after the original expires,
  without re-initiating the entire authorization flow (such as authorization
  code or password grant). It's typically used in long-lived sessions.

- JWT Bearer Grant: A flow defined in RFC 7523 that allows a client holding a
  valid JWT (assertion) issued by a trusted identity provider to exchange it for
  an OAuth 2.0 access token. Useful for federated authentication across trust
  boundaries or in service-to-service (machine-to-machine) interactions.

- Implicit Grant: An OAuth 2.0 flow optimized for single-page applications
  (SPAs) or browser-based clients where the access token is returned directly in
  the URL fragment without an intermediate code exchange. This flow is now
  largely deprecated in favor of Authorization Code with PKCE due to security
  concerns (e.g., token leakage).

- Token Exchange: A protocol extension (RFC 8693) that allows one security token
  (e.g., access token, JWT) to be exchanged for another, typically used in
  delegation (acting on behalf of a user) or impersonation (acting as another
  identity) scenarios in distributed systems or microservices.

- SAML 2.0 Bearer Assertion Grant: A flow that allows a client to present a
  valid SAML 2.0 assertion—often issued after SSO login—as a bearer token to an
  OAuth 2.0 authorization server, in order to obtain an access token. This
  facilitates integration between legacy SAML-based identity providers and
  modern OAuth-based APIs.

- Device Authorization Grant: A flow defined in RFC 8628 designed for
  input-constrained devices (e.g., smart TVs, printers) where the user
  authorizes access on a secondary device (like a smartphone or browser) while
  the client polls the token endpoint to obtain the access token once
  authorized.

### OpenID Connect



## Interview Questions

### What's the difference between authentication and authorization?

Authentication proves who the user is. Authorization determines what they're
allowed to do.

### Explain the OAuth2 Authorization Code flow with PKCE

Client redirects to IdP, user logs in and consents, IdP returns code, client
exchanges code plus PKCE verifier for tokens.

### Why use PKCE in Authorization Code Flow?

**PKCE** (pronounced "pixy") is an **extension to the OAuth 2.0 Authorization
Code flow**, designed to make public clients (like Angular or mobile apps) more
secure,especially those that can't safely store a `client_secret`.

The original **Authorization Code Flow** was built for **confidential clients**
(e.g., server-side apps), which could securely store a `client_secret`.

For **public clients** (e.g., SPAs or mobile apps), storing a secret is
insecure. This created a vulnerability where **attackers could intercept the
authorization code** in a man-in-the-middle (MitM) attack and exchange it for an
access token.

PKCE adds a **code verifier + challenge** mechanism that binds the authorization
code to the requesting client.

#### PKCE Flow

- The client generates a **random string** (code verifier).
  - code_verifier: Random string (43–128 chars), stored client-side
- It hashes the code verifier using SHA-256 -> gets **code challenge**.
  - code_challenge: BASE64URL-ENCODE(SHA256(code_verifier))
- Redirects user to auth server with code challenge (`code_challenge`) and
  method (`S256`). The server stores the `code_challenge` temporarily with the
  **authorization code**.
- After login, the server redirects back with an **authorization code**.
- The client exchanges the **authorization code** + **original code verifier**
  to get the access token. The server **validates** the code verifier against
  the stored challenge.

### How do you revoke a JWT before it expires?

Use a server‐side blacklist or change the signing key. Short-lived access tokens
plus refresh tokens minimize risk.

### What is CSRF and how do you prevent it in cookie-based auth?

CSRF tricks a user’s browser into making unwanted requests. Use anti-forgery
tokens, `SameSite=Lax/Strict` cookies, and custom headers.

### What are the key cookie options you should set for security?

Set `HttpOnly`, `Secure`, `SameSite=Lax/Strict`, and use encrypted payloads via
ASP.NET Core data protection.

### How do you handle failed refresh-token attempts in a SPA?

On refresh failure, clear client state, redirect user to login page, and display
an error indicating session expiration.

### What's the difference between OAuth and OpenID Connect?

OAuth2 is for authorization. OpenID Connect (OIDC) builds on top of OAuth2 to
add authentication (via `id_token`). OIDC is what you'd use to log users in.

### How do you protect JWTs from XSS and CSRF?

Store in memory (not localStorage), use SameSite cookies, and CSRF tokens.

### Where should tokens be stored in an Angular SPA?

Access tokens can be stored in memory or `sessionStorage` (to avoid XSS risk).
Never store tokens in `localStorage` or cookies without `HttpOnly` flag due to
security concerns.

---

## https

## ssl

## tls
