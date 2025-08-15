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

---

## OWASP (Open Web Application Security Project)

OWASP stands for **Open Web Application Security Project** — a **nonprofit
foundation** that focuses on improving software security. Its best-known work is
the **OWASP Top 10**, a regularly updated list of the most critical security
risks for web applications.

Think of it as:

- A **security playbook** for developers and security testers.
- A **community-driven** resource — open source, widely recognized by companies.

### Key OWASP Top 10 (2021 version)

| #   | Risk                                     | Example                           | Mitigation                             |
| --- | ---------------------------------------- | --------------------------------- | -------------------------------------- |
| 1   | Broken Access Control                    | Normal user can access admin API  | Role-based auth, policy checks         |
| 2   | Cryptographic Failures                   | Storing passwords in plain text   | Use salted hashing (PBKDF2, bcrypt)    |
| 3   | Injection                                | SQL Injection via search box      | Parameterized queries, ORM             |
| 4   | Insecure Design                          | Missing rate-limits for login     | Threat modeling, secure design reviews |
| 5   | Security Misconfiguration                | Default admin password left       | Harden configs, remove defaults        |
| 6   | Vulnerable & Outdated Components         | Old jQuery with XSS bug           | Regular dependency updates             |
| 7   | Identification & Authentication Failures | Session fixation                  | Secure cookie flags, proper login flow |
| 8   | Software & Data Integrity Failures       | Using unsigned packages           | Verify signatures, integrity checks    |
| 9   | Security Logging & Monitoring Failures   | No alerts on brute force          | Central logging, anomaly alerts        |
| 10  | Server-Side Request Forgery (SSRF)       | App fetches internal AWS metadata | Whitelist outbound requests            |

Perfect! Let’s go **deep dive OWASP Top 10 (2021)** tailored for your
\*\*Angular

- ASP.NET Core + EF Core stack**, with realistic scenarios, triggers, mitigation
  code, and interview Q\&A. I’ll go **one by one\*\* to make it digestible and
  actionable.

### Broken Access Control (A01)

#### How it happens

Users can access resources or perform actions they shouldn’t because **access
rules are missing or misconfigured**.

#### Realistic scenario

- Angular UI hides “Delete User” button for normal users, but the backend API
  **doesn’t check roles**. A normal user can send `DELETE /api/users/1` and
  delete another user.

#### How it’s triggered

- Direct API calls (bypassing UI)
- Manipulating request parameters (IDs, query strings)
- Missing server-side checks

#### Mitigation

**Backend: ASP.NET Core**

```csharp
[Authorize(Roles = "Admin")]
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(int id)
{
    var user = await _context.Users.FindAsync(id);
    if (user == null) return NotFound();
    _context.Users.Remove(user);
    await _context.SaveChangesAsync();
    return NoContent();
}
```

**Angular:**

```typescript
// Only show delete button for admins
<button *ngIf="user.role === 'Admin'" (click)="deleteUser(user.id)">Delete</button>
```

#### Interview Questions

- **How can you prevent users from bypassing UI restrictions?** Always enforce
  **server-side authorization checks** using roles/claims, never rely solely on
  UI controls.

### Cryptographic Failures (A02)

#### How it happens

Sensitive data is stored or transmitted insecurely — plain text passwords, weak
hashing, missing HTTPS.

#### Realistic scenario

- Passwords stored as plain text in database; attacker leaks DB → all user
  accounts compromised.
- JWT token not signed or symmetric key is weak.

#### Mitigation

**ASP.NET Core Identity (EF Core)**

```csharp
var user = new IdentityUser { UserName = model.Email };
var result = await _userManager.CreateAsync(user, model.Password);
// Uses secure hashing automatically (PBKDF2)
```

**Angular / HTTPS**

```typescript
// Always call APIs over HTTPS
this.http.post("/api/login", credentials, { withCredentials: true });
```

#### Interview Questions

- **What hashing algorithm should you use for passwords?** Use strong, salted
  hashes like **PBKDF2, bcrypt, or Argon2**; avoid SHA1/MD5.

### Injection (A03)

#### How it happens

Untrusted input is executed as code or query. Common types: SQL Injection,
Command Injection, NoSQL Injection.

#### Realistic scenario

- Search box passes `searchTerm` directly to raw SQL:

  ```csharp
  var users = _context.Users
      .FromSqlRaw($"SELECT * FROM Users WHERE Name LIKE '%{searchTerm}%'")
      .ToList();
  ```

  → attacker inputs: `' OR 1=1 --` → returns all users.

#### Mitigation

**EF Core (Parameterized Queries / LINQ)**

```csharp
var users = _context.Users
    .Where(u => u.Name.Contains(searchTerm))
    .ToList();
```

#### Interview Questions

- **How does EF Core protect against SQL injection?** LINQ queries and
  parameterized SQL automatically escape inputs, preventing injection.

### Insecure Design (A04)

#### How it happens

Security is **not considered in architecture**. No threat modeling, missing
rate-limits, weak password policies.

#### Realistic scenario

- API allows unlimited login attempts → brute force attacks succeed.
- No account lockout → attacker guesses passwords.

#### Mitigation

**ASP.NET Core (Identity + Lockout)**

```csharp
var result = await _signInManager.PasswordSignInAsync(
    user.Email, password, isPersistent: false, lockoutOnFailure: true
);
```

**Angular:**

- Disable login button temporarily after multiple failed attempts; show CAPTCHA.

#### Interview Questions

- **How do you ensure secure design in apps?** Threat modeling, secure defaults,
  rate-limiting, input validation, and password policies.

### Security Misconfiguration (A05)

#### How it happens

Default passwords, unnecessary services, verbose error messages, or
misconfigured headers.

#### Realistic scenario

- ASP.NET Core shows full exception stack trace → reveals DB schema or secrets.
- Static files directory exposes sensitive config files.

#### Mitigation

**ASP.NET Core**

```csharp
app.UseExceptionHandler("/error"); // hides detailed info in production
app.UseHsts(); // enforce HTTPS
```

#### Interview Questions

- **How to avoid misconfiguration risks?** Harden servers, disable dev features
  in production, remove default accounts, set secure headers.

### Vulnerable & Outdated Components (A06)

#### How it happens

Using old libraries with known CVEs.

#### Realistic scenario

- Using Angular 10 with a vulnerable RxJS version → attackers exploit known XSS
  bug.
- EF Core 3.x has known vulnerabilities; attacker can escalate privilege.

#### Mitigation

- Regular dependency checks (`npm audit`, `dotnet list package --vulnerable`)
- Update to latest stable versions.

#### Interview Questions

- **How to keep dependencies secure?** Continuous updates, vulnerability
  scanning, monitoring CVEs.

### Identification & Authentication Failures (A07)

#### How it happens

Weak passwords, missing MFA, predictable session IDs, improper logout.

#### Realistic scenario

- JWT token never expires → stolen token usable indefinitely.
- Passwords have no complexity → brute-force attacks succeed.

#### Mitigation

**ASP.NET Core JWT**

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => {
        options.TokenValidationParameters = new TokenValidationParameters {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ClockSkew = TimeSpan.Zero,
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(configuration["JWT:Secret"])
            )
        };
    });
```

**Angular:**

- Store JWT in memory or HttpOnly cookie, **not localStorage**.

#### Interview Questions

- **How can you make JWTs more secure?** Short expiration, HttpOnly cookies,
  refresh tokens, secure signing keys.

### Software & Data Integrity Failures (A08)

#### How it happens

Unsigned packages or libraries, missing verification.

#### Realistic scenario

- Angular app loads JS from CDN without SRI → MITM injects malicious script.
- NuGet package compromised → backend code executes malicious logic.

#### Mitigation

```html
<script
  src="https://cdn.example.com/lib.js"
  integrity="sha384-..."
  crossorigin="anonymous"
></script>
```

- Validate package signatures when installing dependencies.

#### Interview Questions

- **How to ensure third-party packages are safe?** Use signed packages, checksum
  verification, SRI for client-side scripts.

### Security Logging & Monitoring Failures (A09)

#### How it happens

No logs for critical actions → attacks go undetected.

#### Realistic scenario

- Failed login attempts, privilege escalations, or data exfiltration happen with
  no monitoring.

**Mitigation (ASP.NET Core + Serilog)**

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.File("logs/log.txt")
    .CreateLogger();

Log.Information("User {UserId} failed login at {Time}", userId, DateTime.UtcNow);
```

#### Interview Questions

- **Why is logging important?** Helps detect attacks early and aids incident
  response.

### Server-Side Request Forgery (SSRF) (A10)

#### How it happens

Server fetches URLs from untrusted input → attacker accesses internal resources.

#### Realistic scenario

- API fetches images: `/api/fetch?url=http://internal-service/admin` → attacker
  reads internal metadata.

#### Mitigation

```csharp
// Only allow whitelisted domains
var allowedHosts = new [] { "example.com", "images.example.com" };
var uri = new Uri(requestedUrl);
if (!allowedHosts.Contains(uri.Host))
    return BadRequest("Invalid host");
```

#### Interview Questions

- **How can SSRF be prevented?** Validate URLs, whitelist hosts, avoid fetching
  internal resources based on user input.

## OWASP Top 10 Cheat Sheet (2021)

| #       | Risk                                     | Trigger / Scenario                                       | Mitigation                                                        | Sample Code                                                                                           | Interview Q\&A                                                                                                          |
| ------- | ---------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **A01** | Broken Access Control                    | User sends `DELETE /api/users/1` despite not being admin | Enforce server-side role/claim checks                             | **ASP.NET Core:** `[Authorize(Roles="Admin")]` on API<br>**Angular:** `*ngIf="user.role==='Admin'"`   | Q: How prevent users bypassing UI restrictions?<br>A: Always enforce **server-side authorization**, never trust UI only |
| **A02** | Cryptographic Failures                   | Storing passwords as plain text, weak JWT signing        | Use strong hashing (PBKDF2, bcrypt) and HTTPS                     | **ASP.NET Core Identity:** `_userManager.CreateAsync(user, password)`<br>**Angular:** HTTPS API calls | Q: Recommended password hashing? <br>A: PBKDF2, bcrypt, Argon2; never SHA1/MD5                                          |
| **A03** | Injection                                | Search box input directly used in SQL → `' OR 1=1 --`    | Use parameterized queries / ORM                                   | **EF Core LINQ:** `Where(u => u.Name.Contains(searchTerm))`                                           | Q: How EF Core prevents SQL Injection?<br>A: Parameterized queries and LINQ escape inputs                               |
| **A04** | Insecure Design                          | Unlimited login attempts → brute force                   | Threat modeling, secure defaults, rate-limiting                   | **ASP.NET Core Identity:** `lockoutOnFailure: true`                                                   | Q: How ensure secure design?<br>A: Threat modeling, password policies, rate-limits, input validation                    |
| **A05** | Security Misconfiguration                | Dev exception pages visible in production                | Use production error handling, secure headers, remove defaults    | `app.UseExceptionHandler("/error"); app.UseHsts();`                                                   | Q: Avoid misconfiguration risks?<br>A: Harden servers, disable dev features in production, set secure headers           |
| **A06** | Vulnerable & Outdated Components         | Using old Angular / EF Core libraries with CVEs          | Regular updates, vulnerability scans                              | `npm audit`, `dotnet list package --vulnerable`                                                       | Q: Keep dependencies secure?<br>A: Continuous updates, monitor CVEs, scan dependencies                                  |
| **A07** | Identification & Authentication Failures | JWT never expires, weak passwords                        | Short-lived tokens, MFA, secure storage                           | **ASP.NET Core JWT:** `ValidateLifetime = true`, HttpOnly cookie                                      | Q: Secure JWT handling?<br>A: Short expiry, HttpOnly cookies, refresh tokens, secure signing                            |
| **A08** | Software & Data Integrity Failures       | Unsigned JS/CDN scripts → MITM injection                 | Use signed packages, Subresource Integrity (SRI)                  | `<script src="lib.js" integrity="sha384-..." crossorigin="anonymous"></script>`                       | Q: Ensure third-party packages are safe?<br>A: Signed packages, checksum verification, SRI for client scripts           |
| **A09** | Security Logging & Monitoring Failures   | No logs for failed login attempts                        | Centralized logging, alerting                                     | **Serilog:** `Log.Information("Failed login for {User}", userId)`                                     | Q: Why logging important?<br>A: Detect attacks early, support incident response                                         |
| **A10** | Server-Side Request Forgery (SSRF)       | User input URL fetches internal metadata                 | Validate URLs, whitelist hosts, avoid fetching internal resources | `if(!allowedHosts.Contains(uri.Host)) return BadRequest();`                                           | Q: Prevent SSRF?<br>A: Whitelist, validate URLs, avoid internal resource fetch from user input                          |
