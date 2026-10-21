---
name: Security Audit
description: Use when work touches authentication, authorization, sessions, permissions, tokens, secrets, credentials, input validation, file uploads, SQL or shell queries built from user input, redirects, webhooks from outside, or personal data — and before shipping any of it. Triggers on "is this secure", "security review", "audit this", a new auth or login or payment flow, an endpoint reading a client-supplied user or tenant id, adding a dependency, exposing an API, or reviewing a diff that touches any of the above. Use it before the change ships, not after.
---

# Security Audit

Evidence-based. A finding without a reachable path is speculation, not a vulnerability.

## Scope

Establish the trust boundary and the attacker before looking for anything.

- What is untrusted here: request input, headers, cookies, query strings, uploads, webhooks, third-party responses, environment variables, database contents written earlier by a user?
- Who is the attacker, and what do they already control?
- What is the asset worth protecting?
- Which boundary is being crossed?

Most real findings come from answering these four questions, not from a checklist.

## Authentication and sessions

- Passwords hashed with a slow, salted algorithm (argon2, scrypt, bcrypt). Never MD5, SHA1, or plain SHA256 for passwords.
- Session tokens generated from a cryptographically secure source, and compared in constant time.
- Cookies: `HttpOnly`, `Secure`, `SameSite` set deliberately.
- Session invalidation on logout and on privilege change.
- Tokens that expire, and expiry that is actually enforced.
- Password reset tokens single-use and short-lived.
- Rate limiting on login, reset, and verification endpoints.
- No user enumeration: identical responses and timing for existing and non-existing accounts.
- MFA secrets and recovery codes treated as credentials.

## Authorization

This is where most real breaches live. Check every handler, not just the middleware.

- Is the check inside the handler, not only in routing?
- Is it applied to **every** action, including reads, exports, search, and bulk endpoints?
- Is the object being authorized, or only the route? Object-level checks are required: never trust an ID from the client without verifying ownership.
- Can the check be bypassed by changing method, path, content type, or an alternate route?
- Are server-side decisions made from a client-supplied role, user id, tenant id, or price?
- Is there a default-deny path when no rule matches?
- Do bulk and nested operations authorize each item, or just the parent?

Verify by reading each access path. Do not infer coverage from the presence of an auth middleware.

## Injection

- **SQL:** parameterized queries. Flag string concatenation or interpolation into a query, including in ordering and table names where those cannot be parameterized.
- **Command:** no shell invocation with user input. Prefer argument arrays over `sh -c`. Flag `exec`, `eval`, `child_process` with interpolation.
- **Template:** context-aware auto-escaping. Look for `|safe`, `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, `render_template_string`.
- **Path:** user input in a file path can escape the root with `../`. Check normalization happens before the check, and symlinks are resolved.
- **Deserialization:** no untrusted input into a parser that executes or instantiates types, such as `pickle`, `yaml.load`, Java native deserialization, or `unserialize`.

## Input handling

- Validation on the server, not only in the client. Client validation is a convenience, not a control.
- Allowlist over denylist for anything that names a resource, field, or URL.
- Bounds on size: string length, array length, upload size, nesting depth, page size.
- Timeouts and size caps on outbound requests and uploads.
- Redirect targets validated against an allowlist.
- File uploads: validate type by content, not by extension or client-supplied MIME. Store outside the web root. Never use the client filename on disk.

## Secrets

- No secrets in source, comments, tests, fixtures, or logs.
- No secrets in the diff, even in a test meant to be deleted.
- Real secrets come from the environment or a secret manager, not committed files.
- `.env` is ignored by git. Verify rather than assume.
- Redact tokens, authorization headers, and personal data from any error message or log line.
- Rotate rather than reuse when a secret was exposed, and treat it as compromised the moment it landed in a log or a commit.

## Dependencies

```sh
<the project's audit command>       # discover it, do not invent it
```

- Check for known vulnerabilities in direct dependencies first; transitive ones are often noise.
- Check install scripts and lifecycle hooks in newly added packages.
- Confirm the lockfile actually changed and pins versions.
- Note whether a dependency is still maintained.
- A critical advisory with no fix is a finding, and the options are upgrade, replace, or mitigate and document. Do not silently accept it.

## Data exposure

- Responses return only fields the caller needs. No whole ORM or user objects serialized by accident.
- Stack traces and internal paths never reach the client.
- Personal data minimized, and retention bounded.
- Backups, logs, and error reports do not contain more than production does.
- Cross-tenant data is not reachable through any query path.

## Output shape

```text
<verdict>: <one line>

Findings, severity order
- <ID> <title> — <severity>
  Location: <path>:<line>
  Path: <attacker input> → <sink>
  Impact: <what the attacker achieves>
  Evidence: <the code that proves it>
  Fix: <smallest correction>

Checked and clean: <areas examined, so coverage is visible>
Unverified: <what could not be checked and why>
```

## Rules

- Every finding needs a reachable path from untrusted input to the sink. If you cannot describe it, move it to `Unverified`.
- Rate severity by impact and reachability, not by how alarming it sounds.
- Never invent a vulnerability to appear thorough. An empty report on sound code is a valid and useful result.
- Never invent product rules: authentication requirements, retention periods, roles, and compliance obligations are the user's decisions. If one is missing, name it as an open question rather than assuming a policy.
- Prefer specific, minimal fixes. Do not propose a rewrite.
- Do not write exploit payloads aimed at systems you were not asked to test.