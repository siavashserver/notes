---
title: Security
---

## Hashing Methods

Hash functions are mathematical algorithms that transform an input of arbitrary
length into a fixed-size output, known as a hash value or digest. The process is
fundamentally one-way—meaning it is computationally infeasible to reverse the
hash and retrieve the original input.

Hashing is distinct from encryption, serving purposes of data integrity and
authentication rather than confidentiality. Hashes are integral for digital
signatures, password storage, file integrity verification, and blockchain record
linking.

| Algorithm | Year Introduced | Digest Size             | Construction    | Rounds | Collision Resistance | Status                 | Common Use Cases                             |
| --------- | --------------- | ----------------------- | --------------- | ------ | -------------------- | ---------------------- | -------------------------------------------- |
| MD5       | 1992            | 128 bits                | Merkle–Damgård  | 64     | Broken               | Deprecated             | Legacy file integrity (non-secure)           |
| SHA-1     | 1995            | 160 bits                | Merkle–Damgård  | 80     | Broken (since 2017)  | Deprecated             | Legacy HMAC, non-critical integrity          |
| SHA-2     | 2001            | 224, 256, 384, 512 bits | Merkle–Damgård  | 64/80  | Strong               | Recommended            | SSL/TLS, digital signatures, PGP, blockchain |
| SHA-3     | 2015            | 224, 256, 384, 512 bits | Sponge (Keccak) | 24     | Strong               | Recommended            | Emerging cryptography, SHA-2 alternative     |
| bcrypt    | 1999            | 192 bits                | Blowfish-based  | N/A    | Memory/cost tunable  | Recommended (password) | Password hashing (with salt, slow)           |
| scrypt    | 2009            | Variable                | Memory-hard     | N/A    | Memory/cost tunable  | Recommended (password) | Password hashing (high memory usage)         |
| Argon2    | 2015            | Variable                | Memory-hard     | N/A    | Memory/cost tunable  | Recommended (password) | Password hashing (current best)              |

_Table based on common cryptographic literature, NIST publications, and security
best practices._

### MD5

MD5 was once the standard for generating fast file checksums and storing hashed
passwords. It produces a 128-bit digest and is simple, fast, and widely
supported. However, since the early 2000s, practical collision attacks have been
demonstrated, allowing attackers to construct two distinct inputs sharing the
same hash. MD5 is therefore considered cryptographically broken: it is
unsuitable for any application requiring defense against malicious tampering or
attackers—for example, digital signatures, certificates, or password storage.

In its day, MD5’s main advantages were speed and implementation simplicity.
Today, its use is restricted to non-security-critical file integrity checks
where accidental corruption is the main concern, not attacker manipulation.

### SHA-1

SHA-1, the first widely adopted member of the Secure Hash Algorithm family,
produces a 160-bit digest and employs a similar Merkle–Damgård structure as MD5.
In 2017, the “SHAttered” attack demonstrated the first practical collision, and
subsequent computational advancements lowered the cost of generating SHA-1
collisions to practical levels. As a result, SHA-1 is now considered deprecated
for all cryptographic security purposes, and organizations like Google and
Mozilla have retired SHA-1-signed digital certificates.

Transparency mandates that systems still using SHA-1 (for example, in legacy
HMACs) migrate to newer, more resilient algorithms as soon as possible.

### SHA-2 (SHA-224, SHA-256, SHA-384, SHA-512)

SHA-2 is a family of algorithms producing digests of various lengths up to 512
bits. The most widely deployed is SHA-256, which is the backbone for TLS
certificates, blockchain technologies (such as Bitcoin), software signing, and
modern digital signatures. SHA-2 addresses previous vulnerabilities in SHA-1 and
currently suffers from no known practical weaknesses. It is considered the
best-in-class choice for most cryptographic integrity scenarios, barring very
specific requirements for post-quantum or ultra-long-term security needs.

### SHA-3

SHA-3, standardized in 2015, is based on the Keccak sponge construction,
offering resistance to length-extension attacks and flexible digest lengths.
SHA-3 provides an orthogonal security design compared to SHA-2 and serves as a
backup in the cryptographic community in case any weaknesses are discovered in
SHA-2. While its adoption is less widespread, it is particularly valued in
contexts demanding diversity of cryptographic primitives or higher attack
resistance requirements.

### Modern Password Hashing: bcrypt, scrypt, Argon2

General-purpose hashing algorithms like SHA-2 are not suitable for password
storage due to their speed—attackers can brute-force billions of hashes per
second with consumer hardware. Secure systems instead use slow,
resource-intensive password hashing schemes:

- **bcrypt**: Incorporates a salt and work factor/cost parameter, based on
  Blowfish. It is well-tested, widely available, but memory usage is fixed and
  limited.
- **scrypt**: Adds both memory and CPU _hardness_, making attacks using GPUs or
  ASICs uneconomical.
- **Argon2**: Winner of the 2015 Password Hashing Competition. The current gold
  standard for password hashing, Argon2id balances memory/cost with defense
  against both GPU attacks and side-channel leaks. All exploit built-in unique
  salts for each password, ensuring the same password never hashes to the same
  output twice.

### Hashing in Practice

Hashing is used in password storage—servers store only the hash (and salt).
During authentication, the user’s password is hashed with the same salt and
compared to the stored value. Even if the server’s database is compromised,
hashes are computationally difficult to reverse. Modern systems layer salts,
slow algorithms, and sometimes “pepper” (a secret application-wide value) for
additional security.

Hashes are also used for:

- **Data integrity verification**: After downloading software, users compare
  computed hash values (e.g., SHA-256) with published values to confirm
  integrity.
- **Digital signatures**: The sender hashes a message and signs the hash with a
  private key. The recipient verifies both integrity (using the hash) and
  authenticity (via the public key).
- **Blockchains**: Cryptographic hashes link transaction blocks and validate
  state.
- **Message authentication codes (HMACs)**: Combine a secret key and a hash
  algorithm for tamper-proof message authentication.

### Security Weaknesses and Attacks

Hash function attacks commonly include:

- **Collision attacks**: Find two different inputs with the same hash (fatal for
  digital signatures).
- **Preimage attacks**: Attempt to reverse a hash (currently infeasible for
  SHA-2/3).
- **Rainbow tables**: Precomputed databases for reversing unsalted hashes (beat
  by salting).
- **Brute-force/dictionary attacks**: Try popular guesses; mitigated by slow,
  salted hashes and password policies.

### Interview Questions

#### What is the difference between hashing and encryption?

Hashing is a one-way, irreversible process that converts data into a fixed-size
digest. Its primary purpose is data integrity and verification. Encryption is a
two-way, reversible process that transforms plaintext into ciphertext using a
key, and only someone with the key can restore the plaintext. Encryption is
designed to ensure data confidentiality, while hashing is for integrity and
authentication.

#### Why is MD5 considered insecure for cryptographic purposes?

MD5 suffers from practical collision attacks, where attackers can generate two
different inputs with the same hash value. Modern hardware can generate such
collisions in seconds, allowing for digital signature forgery or malicious file
substitution. Because of these weaknesses, MD5 is unsuitable for digital
signatures, certificate signing, and password storage.

#### What is salting, and why is it important in password hashing?

Salting is the practice of adding a unique, random value to each password before
hashing it. This ensures that identical passwords hash to different values,
preventing attackers from using precomputed tables (rainbow tables) and making
brute force attacks harder. Salting is critical for defending against bulk
password compromise events and is implemented automatically in modern password
hashing algorithms like bcrypt, scrypt, and Argon2.

#### Why are slow, memory-hard algorithms (like Argon2, bcrypt) recommended for password storage?

Unlike general-purpose hash functions (SHA-2), slow, memory-hard algorithms
dramatically increase the time and resources required to compute or brute-force
each hash. This thwarts attackers using GPUs and custom hardware, making
large-scale password cracking economically and technologically impractical, even
if the underlying hashes are leaked.

---

## Encryption Methods

### Symmetric Encryption: Advanced Encryption Standard (AES)

AES is the world’s most widely adopted symmetric (secret-key) block cipher. It
operates on 128-bit data blocks, supports key sizes of 128, 192, and 256 bits,
and executes a series of cryptographic operations (substitution, permutation,
mixing, and key addition) over multiple rounds—10/12/14 for 128/192/256 bit
keys, respectively. AES’s structure ensures both confusion (making the
relationship between ciphertext and key complex) and diffusion (spreading
plaintext bits widely).

**Ciphertext** is the output you get when plaintext (readable data) is
transformed by an encryption algorithm using a key (and, in some modes, an IV).

AES replaced the older, insecure DES and 3DES due to its security and
efficiency. Today, AES is embedded in hardware, software libraries, mobile
devices, and is a standard for securing classified and highly sensitive data for
governments worldwide.

#### AES Usage Scenarios

- **Data at rest**: Full-disk encryption software on laptops, cloud storage
  volumes, mobile phones.
- **Data in transit**: TLS (SSL), secure messaging protocols, VPN tunnels.
- **Secure communication**: Wi-Fi WPA2/WPA3, messaging applications (WhatsApp,
  Signal), disk/file encryption.
- **Bulk data encryption**: Database encryption, backup tapes, data warehouses.

#### AES Modes of Operation

AES is a block cipher and requires a mode of operation for encrypting data
streams or files:

- **ECB (Electronic Code Book)**: Deprecated—identical plaintext blocks produce
  identical ciphertext. Not secure for real data.
- **CBC (Cipher Block Chaining)**: Adds randomness (IV), but vulnerable to
  padding oracle attacks without authentication.
- **GCM (Galois Counter Mode)**: Adds authentication (integrity) to encryption,
  high throughput, default for modern TLS.
- **CTR (Counter Mode), OFB, CFB**: Used in streaming or parallelized
  environments.

Best practice is to use AES-GCM or authenticated encryption modes for
communication and storage.

#### AES Encryption with IV — Step-by-Step Flow

1. **Plaintext Input**: The original message or data you want to protect.

2. **Key Selection**: A secret key (128, 192, or 256 bits) is chosen or
   generated securely.

3. **IV Generation**: A random 128‑bit block is created.

   - Purpose: ensures that even if the same plaintext is encrypted twice with
     the same key, the ciphertext will differ.
   - This IV will be sent alongside the ciphertext.

4. **Key Expansion**: The secret key is expanded into multiple round keys for
   the AES rounds.

5. **Initial Transformation**: The plaintext is combined with the IV (in modes
   like CBC, CTR, GCM) before the first encryption step.

6. **AES Rounds**:

   - **SubBytes** → substitute each byte using the S‑box.
   - **ShiftRows** → shift rows to mix data.
   - **MixColumns** → mix columns for diffusion (skipped in final round).
   - **AddRoundKey** → XOR with round key.

7. **Ciphertext Output**: The encrypted data is produced.
   - Sent together with the IV so the receiver can decrypt.

### Asymmetric Encryption: RSA

RSA (Rivest-Shamir-Adleman) is the foundational asymmetric key algorithm for
public-key cryptography. RSA’s security is based on the computational difficulty
of factoring the product of two large primes, known as the modulus. Key pairs
consist of:

- **Public key (n, e)**: Used for encryption or signature verification; openly
  distributed.
- **Private key (n, d)**: Used for decryption or signature creation; kept
  secret.

#### RSA in Practice

- **Public key infrastructure**: Website certificates (HTTPS/TLS), secure email
  (PGP/S/MIME), VPN handshakes, SSH keys.
- **Key exchange**: With RSA, a symmetric session key is securely exchanged,
  after which AES encrypts the session data.
- **Digital signatures**: The sender hashes content, encrypts hash with their
  private key; recipient uses the sender’s public key to verify authenticity and
  integrity.

**Key sizes** range from 2048 bits (current minimum standard, NIST) to 4096 bits
or higher for high-security contexts. RSA decryption is slower than symmetric
encryption, so RSA is seldom used for large data but rather for exchanging
symmetric keys.

**Digital signatures** leverage the reverse: the private key signs (encrypts the
hash), and the public key verifies (decrypts). This provides both message
integrity and authentication.

---

## Comparative Table: Cryptographic Method Use by Use Case

| Scenario                       | Hashing or Encryption  | Algorithm Example       | Security Goal                  |
| ------------------------------ | ---------------------- | ----------------------- | ------------------------------ |
| Password Storage               | Hashing (salted, slow) | Argon2, bcrypt, scrypt  | Persistence, cannot recover    |
| Email/IM Secure Transmission   | Encryption (hybrid)    | RSA/ECC + AES-GCM       | Private, end-to-end secrecy    |
| Digital Signatures             | Hash+Encryption (sign) | SHA-256 + RSA/ECDSA     | Integrity, authenticity        |
| File Integrity Verification    | Hashing                | SHA-2, SHA-3            | Detect tampering or error      |
| Blockchain Transaction Linking | Hashing                | SHA-256                 | Non-repudiable record chain    |
| Disk/Database Encryption       | Encryption (symmetric) | AES-256                 | Protect at-rest information    |
| Secure Web (HTTPS, TLS)        | Hybrid                 | ECDHE+AES-GCM           | Confidentiality, PFS           |
| Backup Tape/Cloud Storage      | Encryption (symmetric) | AES                     | Protect against physical theft |
| Secure VPN                     | Encryption (symmetric) | AES-256, ChaCha20       | Network confidentiality        |
| IoT Device Communications      | ECC Encryption         | ECDH/ECDSA + AES/ChaCha | Efficient comms, small keys    |

---

## Security Best Practices

### Best Practice Checklist for Hashing

| Best Practice                        | Description                                                            |
| ------------------------------------ | ---------------------------------------------------------------------- |
| Use Strong Hash Functions            | Prefer SHA-256, SHA-3, bcrypt, Argon2                                  |
| Salt Password Hashes Uniquely        | Each password + unique salt                                            |
| Memory-Hard Algorithms for Passwords | Use Argon2, scrypt for brute-force defense                             |
| Avoid MD5/SHA-1                      | Never use in new deployments                                           |
| Audit Regularly                      | Identify and migrate away from deprecated                              |
| Use HMAC for Authentication          | Secure message integrity with a secret HMAC                            |
| Apply Layered Defense                | Combine hash methods with peppering/system-wide pepper stored securely |
| Follow NIST and OWASP Guidance       | Maintain compliance and security rigor                                 |

### Best Practice Checklist for Encryption

| Best Practice                        | Description                                                         |
| ------------------------------------ | ------------------------------------------------------------------- |
| AES-256/GCM for Sensitive Data       | Authenticated encryption mode                                       |
| RSA-2048/3072 or ECC-256+            | Public key infrastructure, digital signatures                       |
| Key Management                       | Use HSM/vaults, rotate/retire keys regularly                        |
| No Key Reuse                         | Use separate keys for encryption, signatures, etc.                  |
| Secure Key Generation                | High-quality hardware RNG only                                      |
| Encryption-in-Transit and At Rest    | Implement at file, disk, session, network levels                    |
| Authentication and Integrity         | Use GCM, HMAC, digital signatures for all comms                     |
| Avoid Custom/Non-standard Algorithms | Always standard, publicly-reviewed cryptography                     |
| Stay Updated                         | Respond to new vulnerabilities, quantum transition                  |
| Quantum-Safe Planning                | Identify and migrate long-lived assets to post-quantum where needed |

### Key Management Best Practices

- Keep strict inventory of all cryptographic keys.
- Enforce least privilege access to keys (role-based controls).
- Use automated, audited processes for key rotation, backup, and revocation.
- Destroy obsolete keys securely (NIST SP 800-57 compliance).
- Monitor and log all key accesses; respond promptly to signs of compromise.
- Never store keys unencrypted or in application source code.

### Interview Questions

#### Describe the difference between symmetric and asymmetric encryption.

Symmetric encryption uses a single shared key for both encryption and decryption
(e.g., AES). Asymmetric encryption uses a key pair: a public key for encryption
and a private key for decryption (e.g., RSA, ECC). Symmetric ciphers are
typically much faster and used for bulk data, whereas asymmetric ciphers are
used for key exchange, signatures, or limited small-data encryption due to their
computational intensity.

#### What is a digital signature, and how is it constructed?

A digital signature is created by hashing a message and then encrypting the hash
with the sender’s private key (using RSA or ECDSA). The recipient decrypts the
signature with the sender’s public key and hashes the message. If the two hashes
match, it confirms both the message’s authenticity (only the private key holder
could have signed it) and its integrity (the content hasn’t changed). This
provides non-repudiation, integrity, and authentication.

#### How does AES encryption work? List at least three real-world use cases.

AES processes 128-bit blocks through a series of substitution, permutation, and
mixing rounds, using keys of 128, 192, or 256 bits. Each round transforms the
data based on key-derived values, providing confusion and diffusion. Three
real-world uses: (1) Disk encryption (BitLocker, FileVault); (2) Securing
network communications (TLS/HTTPS, VPNs); (3) encrypting files in cloud storage
(AWS S3, Google Cloud KMS).

### What is the role of a Certificate Authority (CA) in public-key infrastructures?

A Certificate Authority is a trusted third party that verifies and vouches for
the ownership of public keys by issuing digital certificates. CAs bind public
keys to entity identities (domains, organizations, persons). Web browsers and
operating systems trust root CAs, enabling the trust model behind HTTPS, S/MIME,
and code signing.

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

> `withCredentials`: This option primarily affects requests made to a different
> origin (different protocol, domain, or port) than the Angular application is
> served from. For same-origin requests, `withCredentials` has no effect as
> credentials are sent by default. When `withCredentials: true` is used, the
> server must explicitly allow credentials by including the
> `Access-Control-Allow-Credentials: true` header in its response. If this
> header is missing or set to false, the browser will block the response and a
> CORS error will occur.

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
