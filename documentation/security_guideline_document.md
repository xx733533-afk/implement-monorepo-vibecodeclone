# Security Guidelines for `implement-monorepo-vibecodeclone`

This document captures the essential security principles and actionable controls tailored for the `implement-monorepo-vibecodeclone` monorepo. It is intended to guide developers, DevOps engineers, and security reviewers in building and maintaining a secure, resilient multi-service platform.

---

## 1. Introduction

- **Scope**: Covers the frontend (`apps/web`), backend API (`apps/server`), shared packages (`packages/*`), containerization, CI/CD pipelines, and infrastructure.
- **Goal**: Ensure a defense-in-depth posture, secure defaults, least-privilege access, and consistent handling of all untrusted inputs.

---

## 2. Authentication & Access Control

### 2.1. User Authentication

- Use **Better Auth** (or equivalent) with:
  - Secure password hashing (Argon2 or bcrypt) and per-user salts.
  - Enforced password complexity and rotation policies.
  - Account lockout or progressive delays on repeated failed logins.

### 2.2. Session & Token Management

- Issue **HTTP-only**, **Secure**, **SameSite=Strict** cookies for sessions.
- Implement **idle** and **absolute** timeouts; require re-authentication after expiry.
- Protect against session fixation by regenerating session IDs on login.
- If using JWTs:
  - Reject tokens signed with `alg: none`.
  - Validate `exp`, `iat`, and `aud` claims.
  - Use strong symmetric (HS256+) or asymmetric (RS256) keys stored in a secrets manager.

### 2.3. Authorization & RBAC

- Define clear roles (e.g., `admin`, `developer`, `viewer`) and grant minimal permissions.
- Enforce server-side authorization checks on every endpoint.
- Do not trust client-side role flags.
- Validate permissions on shared library flows (e.g., CLI execution, AI generation).

---

## 3. Input Handling & Processing

### 3.1. Prevent Injection Attacks

- **SQL/ORM**: Use Prisma’s parameterized queries or Drizzle prepared statements.
- **Command Injection**: Sanitize and whitelist CLI command names, arguments, and options.

### 3.2. API Input Validation

- Adopt a schema-first approach (e.g., Zod or Joi) for all REST and WebSocket payloads.
- Reject requests with extra or unknown fields.

### 3.3. File Uploads & Container Sandboxing

- Validate file types, size limits, and content signatures (MIME checks).
- Store uploads **outside** webroot with randomized filenames.
- In `cli-manager`, run each command inside a **read-only**, resource-limited Docker container:
  - CPU, memory, and disk quotas.
  - Disable privileged mode and mount only whitelisted volumes.
  - Drop all Linux capabilities except the bare minimum.

---

## 4. Data Protection & Privacy

### 4.1. Encryption

- **In transit**: Enforce HTTPS/TLS 1.2+ with HSTS on both frontend and API.
- **At rest**: Enable AES-256 encryption for database disks and object storage.

### 4.2. Secrets Management

- Store DB credentials, JWT keys, and Docker registry tokens in a secrets manager (e.g., Vault, AWS Secrets Manager).
- Avoid checking secrets into Git; use environment variable injection at runtime.

### 4.3. Logging & Information Exposure

- Sanitize logs to avoid leaking PII, credentials or detailed stack traces.
- Mask sensitive fields (passwords, tokens) before writing to logs.
- Configure centralized log collection with access controls and retention policies.

---

## 5. API & Service Security

### 5.1. Transport & Encryption

- Require TLS for all service-to-service communication (e.g., web ↔ server, server ↔ database).
- Use mTLS where feasible for internal service calls.

### 5.2. Rate Limiting & Throttling

- Leverage Redis to enforce per-user and per-IP rate limits on sensitive endpoints (`/api/ai/generate`, `/cli/run`).
- Return generic error messages on throttling to avoid information leakage.

### 5.3. CORS & CSRF

- CORS: Allow only trusted origins; avoid wildcard (`*`).
- CSRF: Use synchronizer tokens or double-submit cookies for state-changing operations in the frontend.

### 5.4. API Design

- Follow RESTful conventions: use proper HTTP verbs and status codes.
- Version all public endpoints (`/v1/cli/*`), deprecate old versions gracefully.
- Implement health and metrics endpoints behind authentication or IP whitelists.

---

## 6. Web Application Security Hygiene

### 6.1. XSS & Content Security Policy (CSP)

- Escape or sanitize all user-supplied content before rendering.
- Implement a strict CSP header:
  ```
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'sha256-...'; img-src 'self' data:;
  ```

### 6.2. Security Headers

- Strict-Transport-Security: `max-age=63072000; includeSubDomains; preload`
- X-Frame-Options: `DENY`
- X-Content-Type-Options: `nosniff`
- Referrer-Policy: `no-referrer-when-downgrade`

### 6.3. Cookie Security

- Set `HttpOnly`, `Secure`, and `SameSite=Strict` on session cookies.

---

## 7. Infrastructure & Configuration Management

### 7.1. Container Hardening

- Base images: pick minimal, actively maintained distros (e.g., Alpine, Debian Slim).
- Scan images for CVEs (e.g., using Trivy or Clair) and rebuild on new patches.
- Drop unnecessary packages and disable SSH inside containers.

### 7.2. Network Segmentation

- Expose only required ports (e.g., 443, 5432 internally) via Docker Compose networks.
- Place DB and Redis services on an internal network inaccessible from the public internet.

### 7.3. Secrets & Environment Variables

- Use tools like Docker Secrets or Kubernetes Secrets for runtime injection.
- Avoid storing `.env` files in the repo; provide `.env.example` instead.

### 7.4. Configuration Drift & Immutable Infrastructure

- Treat containers as immutable; rebuild rather than patch in place.
- Use Infrastructure as Code (Terraform, CloudFormation) with version control for reproducible environments.

---

## 8. Dependency Management

- Maintain lockfiles (`pnpm-lock.yaml`) for reproducible builds.
- Run automated SCA scans (e.g., Dependabot, Snyk, Renovate) on all packages.
- Remove unused packages and limit transitive dependencies.
- Upgrade to patched versions promptly when CVEs are reported.

---

## 9. CI/CD & DevOps Security

### 9.1. Workflow Hardening

- Restrict GitHub Actions to run only on approved branches.
- Require signed commits or GPG-signed tags for production releases.
- Avoid embedding secrets in workflow YAML; use Actions secrets.

### 9.2. Pipeline Security

- Use `--filter` to limit test scope but always run full security checks before a release.
- Include automated linting, type-checking, SCA scanning, container image scanning, and vulnerability audits.
- Fail the pipeline on any security or scanning error.

### 9.3. Deployment Controls

- Enforce approval gates for production deployments.
- Use canary or blue-green deployments to minimize risk.

---

## 10. Developer & Review Guidance

- Conduct regular security reviews and threat modeling sessions.
- Document and triage any security findings in a centralized issue tracker.
- Provide clear onboarding for new developers on secure coding standards.
- Update this guideline periodically to reflect new threats and best practices.

---

By adhering to these guidelines, the `implement-monorepo-vibecodeclone` project will achieve a robust security posture, ensuring trust, privacy, and resilience as it evolves into the `vibecode-clone` platform.
