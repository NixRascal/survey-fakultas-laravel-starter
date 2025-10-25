# Security Guidelines for the Faculty Survey Application

This document outlines security best practices and recommendations tailored to the **Next.js 14 + TypeScript + Tailwind CSS + Drizzle ORM** faculty survey application starter. It aligns with core security principles—Security by Design, Least Privilege, Defense in Depth—and addresses authentication, data protection, input handling, API security, infrastructure, and DevOps.

---

## 1. Security by Design & Secure Defaults

- Adopt a threat-modeling mindset from the start. Document critical assets (e.g., respondent data, admin credentials) and trust boundaries (public pages vs. admin panel).  
- Initialize all configurations with the most restrictive settings. Enable HTTPS enforcement and HSTS by default.  
- Use environment variables (`.env`) for all secrets—never commit API keys, database credentials, or private keys to the repository.

## 2. Authentication & Access Control

### 2.1 Admin Authentication
- Leverage Better Auth’s built-in session management. Ensure session tokens:  
  - Are stored in secure, `HttpOnly` cookies  
  - Have both idle and absolute timeouts  
  - Regenerate upon privilege changes (prevent session fixation)  
- Enforce strong passwords: minimum length 12, complexity (upper, lower, digits, symbols), and rate-limit sign-in attempts.

### 2.2 Role-Based Access Control (RBAC)
- Extend the `users` schema with a `role` field (e.g., `super-admin`, `admin`).  
- Implement server-side middleware to gate protected routes (`/dashboard`, `/api/*`) based on role.  
- Principle of Least Privilege: grant only required permissions (e.g., only `super-admin` can manage other admins).

### 2.3 Multi-Factor Authentication (MFA)
- Consider integrating an MFA factor (TOTP or SMS) for `super-admin` accounts or any high-risk actions (user management, schema migrations).

## 3. Input Handling & Validation

### 3.1 Server-Side Validation
- Use a schema-validation library (e.g., Zod) for all API routes and Server Actions.  
- Validate: data types, string lengths, allowed enums, numeric ranges.  
- On failure, return standardized error responses (avoid leaking stack traces).

### 3.2 Prevent Injection Attacks
- Drizzle ORM automatically parameterizes queries. Avoid raw SQL unless necessary, and always use placeholders.  
- Sanitize any user-provided strings before passing to third-party libraries (e.g., the Gemini API).

### 3.3 Cross-Site Scripting (XSS)
- Output-encode user-supplied content in React components. For rich text or open-ended responses displayed in the admin panel, use a library like DOMPurify with strict allow-lists.  
- Set a strong Content Security Policy (CSP) header via Next.js custom server or middleware.

### 3.4 Cross-Site Request Forgery (CSRF)
- For state-changing requests (POST, PUT, DELETE), implement CSRF tokens. Better Auth may provide built-in CSRF protection—validate on each form submission.

## 4. Data Protection & Privacy

### 4.1 Encryption in Transit & at Rest
- Enforce HTTPS (TLS 1.2+) for all traffic.  
- Encrypt database connections (PostgreSQL `sslmode=require`).  
- Encrypt sensitive fields (e.g., API keys, PII) at rest using AES-256 if stored server-side.

### 4.2 Secure Password Storage
- Rely on Better Auth’s secure hashing (bcrypt or Argon2 with unique salts). Verify configuration meets current best practices (e.g., Argon2id).

### 4.3 Protection of PII
- Minimize data collection. Only store respondent identifiers required to enforce “one submission per user.”  
- Mask or redact PII in logs and error messages.  
- Provide a data-deletion workflow to comply with GDPR/CCPA requests.

## 5. API & Service Security

### 5.1 Endpoint Hardening
- Secure all `app/api/*` routes: require authentication for admin endpoints, and rate-limit public survey submission routes.  
- Use HTTP verbs semantically (GET for reads, POST for creation). Verify method on the server side.

### 5.2 Rate Limiting & Throttling
- Implement IP-based throttling on `/api/survey/submit` to defend against spam or DoS.  
- Consider using a middleware (e.g., `express-rate-limit` or Vercel Edge middleware).

### 5.3 CORS Configuration
- If serving the frontend and API from the same domain, disable CORS. If not, restrict `Access-Control-Allow-Origin` to trusted domains only.

### 5.4 Secure Secrets Management
- Store `GEMINI_API_KEY`, database credentials, and encryption keys in a secrets manager (e.g., AWS Secrets Manager, HashiCorp Vault) when in production.

## 6. Frontend Security & Hygiene

- Set security headers:  
  - `Content-Security-Policy` (default-src 'self'; script-src 'self')  
  - `Strict-Transport-Security: max-age=63072000; includeSubDomains; preload`  
  - `X-Frame-Options: DENY`  
  - `X-Content-Type-Options: nosniff`  
  - `Referrer-Policy: no-referrer`  
- Secure Cookies: `Secure`, `HttpOnly`, `SameSite=Strict` for session cookies.
- Avoid storing any sensitive tokens in `localStorage` or `sessionStorage`.
- Use Subresource Integrity (SRI) when importing third-party scripts.

## 7. Infrastructure & Configuration Management

- Harden server OS and disable all unnecessary ports and services.  
- Rotate credentials (database, SSH keys) regularly.  
- Ensure Docker images are built from minimal, up-to-date base images.  
- Disable verbose error reporting in production. Ensure stack traces are not exposed to end users.

## 8. Dependency & Supply Chain Security

- Maintain a lockfile (`package-lock.json`).  
- Regularly run automated dependency scans (e.g., Snyk, Dependabot) to detect known vulnerabilities.  
- Audit and remove unused packages to minimize the attack surface.

## 9. DevOps & CI/CD Security

- Enforce branch protection rules. Require code reviews and passing tests before merging.  
- Store all CI secrets (e.g., deploy keys) in the CI provider’s secure vault.  
- Integrate security linters (ESLint security plugins) and run automated tests on pull requests.

## 10. Logging, Monitoring & Incident Response

- Centralize logs (application, access, errors) in a SIEM or log management system.  
- Monitor for anomalous behavior: multiple failed login attempts, spike in survey submissions, unexpected API usage.  
- Define an incident response plan: detection, containment, eradication, recovery, and post-mortem.

---

By following these guidelines, the faculty survey application will be built with security at its core—protecting both respondents and administrators while maintaining data integrity and privacy. Regularly revisit and update these practices as the application evolves and new threats emerge.