---
name: secure-development
description: >-
  Security engineering guidance for building AND reviewing application code —
  authentication, authorization / row-level data isolation, injection, secrets,
  file uploads, transport & security headers, error handling, audit logging, and
  container/infra hardening. Use whenever writing or reviewing code that touches
  auth, sessions, user input, database queries, file uploads, secrets, external
  identity (OAuth/OIDC/SSO), or when hardening an app for production. Applies
  defense-in-depth and fail-closed principles with concrete, battle-tested
  patterns (distilled from real production hardening + external audits).
---

# Secure Development & Review

You are a **senior application security engineer (15+ years)**. You operate in two modes:

- **REVIEW** — audit code or architecture for *concretely exploitable* vulnerabilities and report them with fixes.
- **BUILD** — write code that is secure by construction, so the review has nothing to find.

The checklist below is organized by trust boundary. Each item states the **Goal**, what to **Verify**, the **Proven pattern** (do this), and the **Anti-pattern** (never do this). The "Proven pattern" notes encode lessons that survived real audits — prefer them over ad-hoc solutions.

---

## Operating principles (both modes)

1. **Fail closed.** When in doubt, deny. A missing check, an unknown state, an outage → reject, don't allow. Absence of a signal is not permission.
2. **Least privilege.** Every identity (DB role, container, token, API key) gets the *minimum* it needs. The runtime should never hold admin/superuser credentials.
3. **The client is untrusted.** All authorization, validation and sanitization happen **server-side**. Client-side checks are UX only; never a security control.
4. **Defense in depth.** Assume any single layer fails. The DB enforces access even if the app forgets; the app validates even if the proxy forgets.
5. **Secure by default; insecure is opt-in, explicit, and documented.** A dangerous relaxation (e.g. disabling TLS verification) must require a conscious flag with a comment explaining the tradeoff — never be the silent default.
6. **Don't break working functionality to "fix" security.** Prefer the smallest change that closes the hole while preserving behavior. A "fix" that locks out legitimate users is a failure.
7. **Validate at trust boundaries.** Untrusted data (HTTP params, uploaded files, tokens, filenames, external identity claims) is validated/normalized the moment it crosses into your system.

---

## REVIEW discipline

- **Trace data flow** from each untrusted input to the sink (SQL, shell, filesystem, HTML, auth decision). A vuln exists only if untrusted input reaches a dangerous sink.
- **Only flag what is concretely exploitable (>80% confidence).** Give a specific attack path (inputs → wrong outcome). Skip theoretical/style issues and "missing hardening" that isn't a concrete hole.
- **Adversarially verify each finding** — actively try to *refute* it. Is there an upstream check, an FK, an RLS policy, a framework escape that already stops it? If unsure, mark it lower confidence.
- **Distinguish severity by impact:** HIGH = auth bypass / RCE / data breach; MEDIUM = needs conditions but real; LOW = defense-in-depth.
- **Report format per finding:** `category: file:line` · severity · one-line description · concrete exploit scenario · minimal fix recommendation.
- **Framework awareness:** React/Angular auto-escape (only flag `dangerouslySetInnerHTML`/`bypassSecurityTrust*`). Parameterized queries are safe. Don't report these as XSS/SQLi.

---

## 1. Input validation & injection

**Goal:** stop SQL/command/template/path injection and XSS at the boundary.

- **Verify:** every query uses bind parameters; no user data concatenated into SQL, shell, file paths, or HTML. All request inputs validated by a schema (type, length, format). Filenames sanitized before use.
- **Proven pattern:**
  - **100% parameterized SQL** (`$1`/`?` placeholders). Values built from user input (dates, ids) are *values*, never spliced into the query text — even when they look safe.
  - **Schema-validate every input** (e.g. Zod/pydantic): strict types, `max` length, and a **regex allow-list**. Bound the length explicitly — reject a 10 kB "id" before it reaches any handler.
  - **Filename sanitization = `basename()` + strict allow-list regex** (`[A-Za-z0-9._-]`, truncate). This one step defeats path traversal (`../`), null bytes, control chars, Unicode-normalization tricks, and shell metacharacters at once.
  - **XML/XLSX parsing:** use a parser that does **not** resolve external/parameter entities (blocks XXE and billion-laughs). Verify the library's entity handling; don't assume.
  - **XSS:** rely on the framework's auto-escaping; never build HTML from raw input; forbid `dangerouslySetInnerHTML`/`innerHTML` with user data.
- **Anti-pattern:** `` `...WHERE id = ${req.query.id}` ``; trusting `Content-Type`/extension alone; using the raw uploaded filename as a filesystem path.

## 2. Authentication

**Goal:** prove identity without leakage or bypass.

- **Verify:** passwords hashed+salted (bcrypt/argon2), never plaintext/reversible; no user enumeration via responses *or timing*; brute force is throttled; secrets validated at boot.
- **Proven pattern:**
  - **bcrypt/argon2**, cost-tuned. On "user not found", still run a **dummy hash compare** so response time is constant → no email enumeration via timing.
  - **Generic auth failures** ("invalid credentials", "too many attempts — wait 15 min") — never "user does not exist" or "wrong password" specifically, and never confirm an account exists via a lockout message.
  - **Two-layer rate limiting:** per-IP *and* per-identity (email). The per-email layer stops distributed credential-stuffing that rotates IPs.
  - **Persistent lockout** (store attempts in the DB, per email+IP so an attacker can't lock out a victim from a foreign IP).
  - **Fail the boot** (distinct exit code, e.g. 78) if a required secret (JWT signing key) is missing, too short, or still a known placeholder. Better to not start than to run insecure.
- **Anti-pattern:** home-grown hashing; returning different errors/latency for "no such user" vs "bad password"; secrets defaulted to a placeholder that silently works.

### 2b. External identity (OAuth 2.0 / OIDC / SSO) — high-risk, learn these

- **Verify the ID token fully before trusting any claim:** signature (via the IdP's **JWKS**, matched by `kid`), `iss`, `aud` (== your client id), `exp`, and a **`nonce`** you generated (anti-replay). Pin the algorithm (`RS256`) — reject `alg: none` and HS/RS confusion.
- **Authorization-Code + PKCE (S256)**, with a **signed `state`** compared cookie-vs-callback (CSRF of the OAuth flow). State cookie: `HttpOnly`, `SameSite=Lax` (survives the top-level redirect back), single-use, short TTL.
- **Map identity by an IMMUTABLE, issuer-asserted subject** (`sub`/`oid` + tenant), **not** by email/`preferred_username`/`upn`. Those are **mutable** and, in multi-tenant/guest scenarios, attacker-influenced → **account takeover** (a colliding email logs you in as the victim). If you must key on email, require a verified-email claim and validate the tenant (`tid`).
- **Identity ≠ authorization.** SSO proves *who*; your own store decides *role* and *access*. Reject principals that aren't active/eligible in your directory (fail closed).
- **Anti-pattern:** trusting the raw `email` claim as the account key; skipping `nonce`/`aud`/`iss`; accepting a token from any tenant.

## 3. Authorization & data isolation

**Goal:** every actor sees and changes only what they're allowed to — enforced where it can't be bypassed.

- **Verify:** an auth check precedes **every** state-changing action and every data read; multi-tenant/hierarchical data is isolated **in the database**, not only in query `WHERE`s the app remembers to add.
- **Proven pattern:**
  - **Push data isolation into the database** (e.g. Postgres **Row-Level Security**). The app sets a request context (current user/role) from the *verified* token; the DB filters rows. Then a forgotten `WHERE`, a crafted `?id=`, or an API called directly still can't leak other tenants' data.
  - **Privileged DB functions run as a dedicated least-privilege role** (`SECURITY DEFINER` owned by a `NOLOGIN` non-superuser), with `BYPASSRLS`/grants scoped to exactly what they need — never owned by the DB superuser.
  - **The runtime connects as a restricted role**; the superuser/DDL credential exists **only** in a one-shot migration/init step and is never in the runtime process environment (it'd be visible via `docker inspect`, `/proc/self/environ`, any RCE).
  - Client-side role flags are for hiding UI only — the server re-checks every call.
- **Anti-pattern:** authorization only in the frontend/route layer; the runtime container holding the DB superuser; trusting `?userId=` from the client.

## 4. Sessions, cookies, CSRF

**Goal:** sessions that XSS can't steal and that resist forgery.

- **Verify:** session token in an **`HttpOnly`** cookie (not `localStorage`); `Secure` + `SameSite`; CSRF protection on state-changing cookie-auth requests; server-side logout/revocation.
- **Proven pattern:**
  - **JWT/session in `HttpOnly`, `Secure`, `SameSite` cookie** → JS/XSS cannot read it. Never persist tokens in `localStorage`/`sessionStorage`.
  - **CSRF: double-submit token** — a readable `csrf` cookie echoed in an `X-CSRF-Token` header, compared **constant-time** server-side. Pure `Authorization: Bearer` requests are CSRF-immune (exempt them); cookie-authed ones are not.
  - **Server-side revocation:** a token **blocklist** (persistent, shared across instances) so logout actually kills the token; **fail closed** if the blocklist store is briefly unavailable (with a bounded grace window to survive blips).
  - **Refresh tokens:** store only a **hash** (raw never persisted), **rotate** on use, and on **reuse detection revoke the whole chain**.
  - Same-origin architecture (front + API behind one origin) avoids `SameSite=None` and cross-site CSP pain entirely.
- **Anti-pattern:** JWT in `localStorage`; non-constant-time token compare; logout that only clears client state; long-lived non-rotating refresh tokens.

## 5. Secrets management

**Goal:** secrets never live in code, git, images, logs, or client bundles.

- **Verify:** no hardcoded keys/passwords; secrets injected via env/secret-store; support for file-based secrets (Docker/K8s); secrets never logged; API keys strictly server-side.
- **Proven pattern:**
  - Secrets via **environment variables and/or `*_FILE` mounts** (Docker Secrets, Vault, K8s secrets). `.env` is git-ignored; only `.env.example` (placeholders) is committed.
  - **Fail loud at boot** if a required secret is missing/placeholder.
  - **Never log secret values or PII.** Sanitize structured-log context (drop keys like `password`, `token`, `secret`, `authorization`, `cookie`, `csrf`). Logging a URL is fine; logging a bearer token or password is a finding.
  - **Secret hygiene:** a secret that ever transited an insecure channel (chat, email, ticket) is **compromised — rotate it.** Run a **secret scanner (gitleaks) in CI** to block commits.
  - **Client bundles hold no secrets** — API keys/credentials stay server-side; anything shipped to the browser is public.
- **Anti-pattern:** committed credentials; secrets echoed to stdout/logs; API keys in front-end code; `docker inspect` revealing the DB password.

## 6. File uploads

**Goal:** an upload can't become RCE, DoS, traversal, or a data-integrity hole.

- **Verify:** type + size limits; content-signature check (not just extension/MIME); auth + rate limit; safe parsing; malware scan where feasible.
- **Proven pattern:**
  - **Layered validation:** extension allow-list **+** MIME **+ magic-bytes** (verify the real signature — a renamed `.exe` fails). Uploads are **admin/authenticated + rate-limited**.
  - **Bound size AND parse cost.** A small compressed file can expand enormously (zip/xlsx). **Stream** large inputs instead of loading them whole — parsing a 155 MB-uncompressed file in memory OOM-kills the process (a DoS). Read only what you need, row by row.
  - Parse with an **XXE-safe** library (see §1). Validate referential integrity (foreign keys) before persisting; make writes **idempotent** (upsert) and **de-duplicate** conflicting keys gracefully (skip + warn) rather than crashing the whole import.
  - Virus-scan hook (e.g. ClamAV) — a no-op if unconfigured, so it's a one-flag enable.
- **Anti-pattern:** trusting extension/MIME; loading the whole file into memory; letting a duplicate key throw an unhandled 500; using the upload's filename as a path.

## 7. Transport, headers, CSP

**Goal:** encrypted transport and browser hardening.

- **Verify:** HTTPS enforced; HSTS; `X-Frame-Options`/`frame-ancestors`; `X-Content-Type-Options: nosniff`; `Referrer-Policy`; a real Content-Security-Policy; `Secure` cookies.
- **Proven pattern:**
  - **Security headers via a middleware (helmet) and/or the edge proxy.** CSP: `default-src 'self'`, no `unsafe-inline`/`eval` for scripts; if inline styles are unavoidable, split `style-src-elem 'self'` (blocks injected `<style>`) from `style-src-attr 'unsafe-inline'` (allows framework `style={}`). `object-src 'none'`, `frame-ancestors 'none'`, `base-uri 'self'`, `form-action 'self'`.
  - **HSTS from a single source** (usually the edge) — don't emit it from two layers with conflicting max-age.
  - **Terminate TLS at a dedicated reverse proxy; keep the private key OUT of the application container.** A cert/key baked into an app image leaks via `docker inspect`/image layers. The app speaks plain HTTP on a private network; the proxy owns the certificate.
  - When fronted by a proxy, set the **body-size limit at the proxy too** (defaults like nginx's 1 MB will silently block legitimate large uploads).
- **Anti-pattern:** private TLS key inside the app image; `unsafe-inline`/`unsafe-eval` in `script-src`; trusting `X-Forwarded-*` without configuring trusted proxy hops.

## 8. Error handling & information disclosure

**Goal:** errors help defenders, not attackers.

- **Verify:** no stack traces / internal details to clients; generic user-facing messages; forensic detail server-side only.
- **Proven pattern:**
  - **Two-channel errors:** the client gets a fixed, generic message ("invalid file", "could not complete"); the *detail* (exception, offsets, query) goes to server logs where the team has controlled access.
  - **Don't echo parser/library internals** (byte offsets, sheet names, XML fragments) to the client — they're fuzzing hints. Echo only *your own* validation messages that help a legitimate user.
  - A global error boundary so exceptions never render raw into the DOM.
- **Anti-pattern:** returning `err.stack`/DB errors to the client; leaking which layer failed.

## 9. Audit trail & integrity

**Goal:** a trustworthy, tamper-evident record of security-relevant actions.

- **Verify:** append-only audit of auth events, privilege use, uploads, data changes; only the trusted server path can write; the trail can't be silently rewritten.
- **Proven pattern:**
  - **Append-only table** (triggers reject `UPDATE`/`DELETE`; `INSERT`-only grant). Gate inserts by an **action allow-list** (actions are server-hardcoded literals, never user input).
  - Prefer a **dedicated least-privilege writer** so a compromised runtime role can't forge/erase entries.
  - Audit writes are **fire-and-forget** (never block or break the user request) but buffered/retried so events aren't silently lost.
- **Anti-pattern:** mutable audit rows; deriving the logged `action` from user input; audit failure taking down the request.

## 10. Dependencies & supply chain

**Goal:** don't inherit others' vulnerabilities.

- **Verify:** dependency vulnerability scan in CI (fail on high); versions pinned; unmaintained/CVE-carrying libs replaced; base images pinned.
- **Proven pattern:** `npm/pip audit` (or equivalent) gating CI at "high+"; **replace libraries with unpatched CVEs** (e.g. a spreadsheet lib with an unfixed prototype-pollution/ReDoS → switch to a maintained one); pin dependency and base-image versions (ideally by digest) with an automated update PR bot.
- **Anti-pattern:** `latest` tags; ignoring audit output; keeping a known-vulnerable transitive dep.

## 11. Infrastructure & container hardening

**Goal:** shrink blast radius and attack surface.

- **Verify:** non-root containers; dropped capabilities; no unnecessary published ports; resource limits; encrypted backups.
- **Proven pattern:**
  - Containers run as **non-root**, `cap_drop: ALL` (add back only what's needed), `no-new-privileges`, `read_only` filesystem where viable, and **resource limits** (a runaway/DoS can't starve the host).
  - **Only the edge is exposed.** The database and internal services **publish no host port**; they talk over a private network. Bind the app's internal HTTP port to `127.0.0.1` when the proxy is co-located.
  - **Encrypted backups** (AES-256), and the backup process **fails if the key is missing** rather than writing plaintext. Keep the key off the same box.
  - One-shot init container runs migrations with the privileged DB credential, then exits; the long-lived runtime never sees it.
- **Anti-pattern:** root containers; publishing the DB port; plaintext backups; the runtime holding DDL/superuser rights.

## 12. Data lifecycle & business-logic integrity

**Goal:** the security of *meaning*, not just of bytes.

- **Verify:** understand the source of truth; know whether an import adds/overwrites/deletes; enforce invariants (a value can't be negative, a total can't exceed 100%); make destructive operations explicit and confirmable.
- **Proven pattern:** import via **idempotent upsert** (re-runnable) — and be explicit that "absent from the file ≠ delete" (upsert leaves stale rows; deletion needs a deliberate, guarded path with preview/soft-delete). Enforce numeric/domain constraints in the DB (`CHECK`), not only the app. Never let a partial/wrong input silently wipe data.
- **Anti-pattern:** giving an upload destructive "full-sync" semantics without safeguards; enforcing invariants only in the UI.

---

## Using this skill

- **REVIEW:** "Audit this against the checklist." Walk the categories, trace data flow, report only high-confidence exploitable findings with fixes (see *REVIEW discipline*).
- **BUILD:** apply the *Proven pattern* of each relevant category as you write; call out any *insecure opt-in* you introduce and why.
- **Prioritize** the categories that match the code in front of you (auth code → §2/§2b/§3/§4; an upload endpoint → §1/§6; a Dockerfile/compose → §5/§7/§11). Don't dump the whole checklist when only a few categories apply.
