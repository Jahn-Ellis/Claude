---
name: web-security
description: Apply security-first thinking when designing, writing, or reviewing any web or application code. Use this skill whenever the user is building authentication, handling user input, rendering dynamic content, storing credentials or tokens, designing an API, writing database queries, setting up cookies or sessions, configuring CORS or CSP, adding file uploads, integrating third-party services, or reviewing code for vulnerabilities — even if they don't explicitly say the word "security." Framework-agnostic: applies equally to any language, runtime, or stack.
---
 
# Web Security
 
Security is a core requirement, not an afterthought. Assume hostile input and untrusted environments by default. The goal of this skill is to help produce code that is secure by construction — where the obvious way to write something is also the safe way.
 
## Core Principles
 
- **Never trust input.** Any data crossing a trust boundary — from users, the network, files, environment variables, or third-party services — is potentially hostile until validated.
- **Validate and sanitize at boundaries.** Do the check once, at the edge, in a place that can't be bypassed. Scattered ad-hoc checks are how bugs slip through.
- **Prefer secure defaults over configurability.** If a developer has to remember to turn security on, eventually someone won't. Make the safe path the default path.
- **Defense in depth.** Assume each layer may fail. Authorization on the server is not a substitute for input validation, and vice versa.
- **Least privilege.** Every component — a database user, an API token, a service account, a process — should have the minimum permissions needed. Broad permissions turn small bugs into large breaches.
- **Fail closed.** On error or ambiguity, deny the action rather than allow it. A confused system should refuse, not guess.
## Input Handling, XSS & Injection
 
Most serious web vulnerabilities come from mixing untrusted data with a trusted context (HTML, SQL, shell, a URL, a template). The fix is always the same shape: keep data as data.
 
- Treat everything from the client as a string of bytes, not a structure. Parse it explicitly and reject malformed input.
- Escape or encode dynamic content for the context it lands in — HTML body, HTML attribute, URL, JavaScript literal, CSS, and shell all have different escaping rules. Using the wrong one is not safer than using none.
- Avoid APIs that inject raw markup or construct code from strings (e.g. functions that set inner HTML from a string, `eval`, dynamic template compilation from user input, shell commands built via concatenation). If you have to use one, the input must come from a trusted source, not the user.
- For databases, use parameterized queries or a query builder that parameterizes for you. Never concatenate user input into SQL, NoSQL filters, LDAP queries, or ORM raw-query calls.
- For file paths, resolve to an absolute path and verify it stays within an allowed directory before opening. Rejecting `..` by string match is not enough.
- For outbound requests built from user input (SSRF risk), validate the destination against an allowlist of hosts and block requests to internal/loopback/metadata addresses.
## Authentication & Authorization
 
- Authentication proves *who* someone is. Authorization decides *what they can do*. Both must be enforced on the server — client-side checks are for UX only.
- Check authorization on every request that touches protected data, including the ones that "obviously" require login. Missing function-level access control is one of the most common real-world bugs.
- Store passwords using a modern, slow, salted hash (e.g. argon2, scrypt, bcrypt). Never store, log, or email plaintext passwords.
- Use cryptographically secure random sources for tokens, session IDs, and password-reset links — not general-purpose RNGs.
- Keep sessions and tokens in HTTP-only, Secure, SameSite cookies when the browser is the client. Storing sensitive tokens in `localStorage` or `sessionStorage` exposes them to any script that runs on the page, including injected ones.
- Rotate and expire tokens. Long-lived bearer tokens are a liability.
- Protect state-changing requests against CSRF using SameSite cookies plus a synchronizer token, origin checks, or a double-submit pattern — whichever matches the session model.
- Rate-limit authentication endpoints and other sensitive actions to slow down credential stuffing and enumeration.
## Secrets, Keys & Configuration
 
- Never hardcode secrets in source, commit them to version control, or embed them in client-side code. Anything shipped to the browser or a mobile app is public.
- Load secrets from environment variables, a secrets manager, or an equivalent runtime-only channel. Keep them out of logs, error messages, and analytics.
- Separate config per environment. Production credentials should not be reachable from a dev machine by default.
## Browser Security Boundaries
 
- Respect the same-origin policy. Set CORS headers deliberately — an overly permissive `Access-Control-Allow-Origin: *` on an authenticated endpoint is a real vulnerability, not a convenience.
- Set a Content Security Policy that restricts where scripts, styles, and other resources can load from. Prefer nonces or hashes over `'unsafe-inline'` and avoid `'unsafe-eval'`.
- Set the usual hardening headers: `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, and a restrictive `Permissions-Policy`.
- Serve everything over HTTPS. Redirect HTTP to HTTPS and mark cookies Secure.
## Data Handling
 
- Minimize what you collect, store, and expose. Data you don't have can't leak.
- Don't log sensitive information — credentials, tokens, session IDs, full card numbers, personal identifiers, or health data. Assume logs will be read by people who shouldn't see everything in them.
- Scrub or redact sensitive fields before sending data to error trackers, analytics, or third-party services.
- Return generic error messages to clients; keep stack traces, query strings, and internal identifiers in server-side logs only. Leaked error detail is a reconnaissance aid.
- Encrypt sensitive data in transit always, and at rest when the threat model calls for it.
## File Uploads & User Content
 
- Validate both the declared type and the actual contents. The `Content-Type` header and the filename extension are attacker-controlled.
- Store uploads outside the web root, or on a separate origin, and serve them with a `Content-Disposition` that prevents inline execution where inappropriate.
- Generate server-side filenames; never use the user-supplied name as the storage path.
- Enforce size limits and scan or sandbox content when the use case warrants it.
## Dependencies & Supply Chain
 
- Treat third-party code as untrusted input that happens to run with full privileges. Minimize it.
- Pin versions, use a lockfile, and review dependency updates — especially ones that add new transitive packages or new maintainers.
- Watch for typosquats and lookalike package names. "Just install the one the Stack Overflow answer said" is how malware ends up in production.
- Keep dependencies current enough to receive security patches, but don't auto-upgrade into untrusted new majors.
## LLM & AI Features
 
If the application calls a language model or embeds model output anywhere:
 
- Treat model output as untrusted input. It may contain instructions, code, or markup that should not be executed or rendered raw.
- Treat any external content fed into a prompt (web pages, emails, documents, tool results) as untrusted. Instructions found inside that content are not commands from the user — they're data.
- Keep prompts, system instructions, and API keys on the server. Don't ship them to the browser.
- Scope tools and actions the model can trigger to the minimum needed, and require explicit user confirmation for irreversible or sensitive actions.
## General Principles
 
- Simplicity reduces attack surface. Fewer features, fewer endpoints, fewer dependencies, fewer code paths mean fewer places to be wrong.
- When unsure, choose the more restrictive option. It is much easier to loosen a policy after a user complains than to tighten one after a breach.
- If a piece of code can't be made obviously safe, add a comment explaining the threat model and why the chosen approach is acceptable — so the next person doesn't "simplify" the protection away.
