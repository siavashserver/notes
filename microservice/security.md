---
title: Security
---

## Common attack types

### SQL Injection

SQL Injection occurs when untrusted input is sent to a database interpreter as
part of a SQL statement. Attackers can manipulate queries to access, modify or
delete sensitive data, and even gain full control of the backend database.

Suppose an app builds a SQL query like:

```sql
SELECT * FROM Users WHERE Username = 'admin' AND Password = 'password';
```

If the input isn't sanitized, an attacker could supply `' OR '1'='1` to bypass
authentication altogether.

#### Mitigation Strategies

- Parameterized queries / ORM frameworks (e.g. Entity Framework) avoid
  injection.

- Validate input strictly—use whitelist patterns or types.

- Run databases under least-privileged accounts.

### Cross‑Site Scripting (XSS)

XSS is a type of injection where malicious scripts are embedded into otherwise
trusted websites. These scripts run in users’ browsers, enabling theft of
session data, redirection, or other malicious behavior.

Types of XSS:

- Stored XSS: malicious script is persisted (e.g. in database), and executes
  whenever the content is viewed.

- Reflected XSS: script is reflected in server response (e.g., via URL
  parameters) and executes immediately on page load.

- DOM-based XSS: script runs by manipulating DOM client-side, independent of
  server sanitization.

#### Mitigation Strategies

- Contextual output encoding/escaping: apply appropriate escaping based on
  context (HTML, JavaScript, CSS, URL).

- Use frameworks that automatically encode data (e.g. Razor or Angular
  interpolation).

- Implement Content Security Policy (CSP) with nonces or hashes to restrict
  allowed scripts.

### Cross‑Site Request Forgery (CSRF) / XSRF

CSRF tricks a logged-in user's browser into making unwanted requests to a
trusted site (like submitting a form) because the browser automatically includes
authentication cookies—even if the request originates from an attacker site.

Attacker crafts a hidden form, image tag, or script targeting a sensitive
endpoint. When the victim visits the attacker’s page while authenticated to the
trusted site, the harmful request executes automatically without user consent.

#### Mitigation Strategies

- Synchronizer Token Pattern: Embed a unique, server-generated token in each
  HTML form and validate it on submission. This is the classic anti-forgery
  token approach.

- Double Submit Cookie & Cookie-to-Header:

  - Server sets a CSRF token cookie.
  - JavaScript also reads the cookie and adds it to a custom header on AJAX
    requests.
  - Server verifies header matches the cookie.

- SameSite Cookie Attribute: Tag session cookies with `SameSite=Lax/Strict` so
  they aren't sent with cross-site requests, limiting CSRF exposure.

### Distributed Denial of Service (DDoS)

A DDoS attack is when a network or application is flooded with overwhelming
traffic from multiple, distributed sources, often compromised devices or
botnets, to exhaust resources and deny service to legitimate users.

#### Mitigation Strategies

- Rate Limiting: Throttle requests per IP or client (e.g. 100 req/min per IP) to
  protect against floods and API abuse.
- Firewalls & WAF: Block known malicious IPs, filter Layer 7 patterns, and
  inspect HTTP traffic to stop attack vectors targeting web applications.
- CDN: Consider third-party providers like Cloudflare, Akamai, AWS Shield, or
  others for global-scale absorption and mitigation. These services offer
  protective layers capable of handling attacks in Tbps scale.

---

## Content Security Policy (CSP)

A Content Security Policy is an HTTP header (or meta tag) that lets a website
define exactly which sources of content are safe, such as scripts, styles,
images, iframes, and more. By explicitly listing approved origins, CSP blocks
untrusted inline or external resources, significantly reducing risks like
cross-site scripting (XSS), clickjacking, and code injection.

Key CSP directives:

- `default-src`: fallback for all content types.
- `script-src`: allowed origins for JavaScript.
- `style-src`, `img-src`, `connect-src`, etc.: limits for styles, images,
  AJAX/WebSocket fetches.
- `frame-ancestors`: specifies which frames can embed your page (powerful
  clickjacking defense).

```
Content-Security-Policy:
  default-src 'self'; // by default only load from same origin
  script-src 'self' https://trusted.cdn.com; // load from same origin, and specified cdn
  style-src 'self' 'unsafe-inline'; // allow inline styles
  img-src 'self' data:; // allow embedded images
  frame-ancestors 'none'; // prevent other pages to embed site
```

## HTTP Strict Transport Security (HSTS)

HTTP Strict Transport Security is a response header which tells browsers: for a
set duration, always use HTTPS to connect—and never allow HTTP or bypass TLS
errors. It defends against downgrade (SSL-stripping) attacks and protects
sensitive cookies from being sent in plaintext over HTTP.

**Serve the HSTS header only over HTTPS—if sent via HTTP it’s ignored.**

**Configure your server to respond to all HTTP requests with an HTTP 301
(Permanent) or 302 (Temporary) redirect pointing to the HTTPS version.**

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

---

## Secure password storage

Secure password storage in a database is critical to protect user credentials,
even in the event of database compromise.

### Never Store Plaintext Passwords

Always **hash** passwords before storing. Hash functions are one-way: you can't
reverse-engineer the original password from the stored hash.

### Salting Passwords

- A **salt** is a unique, randomly generated value for each user and each
  password.
- Salts prevent two identical passwords from producing identical hashes, make
  rainbow tables impractical, and force attackers to brute‑force each password
  individually.
- Salt should be generated with a **CSPRNG**, be at least 16 bytes (32–64 bytes
  preferred depending on hash algorithm), and stored in the database alongside
  the hash.

### Hashing with Work Factors

Use **adaptive, slow hashing algorithms** that include built-in salt and allow
tuning of computational difficulty:

- **Argon2 (Argon2id preferred)** – winner of the Password Hashing Competition.
  Configurable for memory, time, and parallelism parameters.
- **bcrypt** – widely available, includes cost factor to increase rounds.
- **PBKDF2** – robust when configured with high iteration count (e.g. 600,000
  for PBKDF2-HMAC-SHA256 per OWASP).

**Avoid fast cryptographic functions** like MD5, SHA-1, or plain SHA-256—they're
too fast and easy to brute-force.

### Peppering (Optional, Extra Layer)

- A **pepper** is a secret value, shared across all passwords but **not stored**
  with the hash.
- Often applied as additional HMAC step after hashing with salt. Stored
  separately (e.g. in a secrets vault or HSM).
- Adds protection if the database is compromised without also leaking the
  pepper.

---

## Secure JWT storage

JWTs typically include an access token (and often a refresh token) that grants
access to protected endpoints. If attackers steal your token, they can
impersonate a user until the token expires—or longer, if able to refresh.

### Common (Unsafe) Storage Options

- **localStorage / sessionStorage**: Easily accessible by JavaScript, making
  them vulnerable to XSS attacks.
- **Storing in JavaScript variables only**: Safer than localStorage, but still
  exposed if XSS is present.
- **Cookie without proper flags**: Cookies accessible via JavaScript
  (`document.cookie`) are still vulnerable to XSS and script taps.

### Recommended Best Practices

#### Use HttpOnly Secure Cookies (Preferred)

- Store JWTs in **HttpOnly cookies** so they're inaccessible to JavaScript.
- Set the **Secure** flag (only sent over HTTPS).
- Set **SameSite=Lax or Strict** to restrict cross-site usage (mitigates CSRF).

Pros:

- Protected from XSS.
- Automatically sent with requests.

Cons:

- Susceptible to CSRF—but mitigated via SameSite and/or anti-CSRF tokens.

#### Use In-Memory Storage + Refresh Logic

- Store JWTs in JavaScript memory (e.g. React or Angular state).
- Do **not** persist tokens in local/sessionStorage.
- On page reloads, either re-acquire tokens or invalidate the session if none
  found.

Pros:

- Eliminates persistent access for attackers after reload.

Cons:

- Resets on refresh—requires reauthentication or silent token refresh.
