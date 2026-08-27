# AGENTS.md
## Add the next section into your AGENTS.MD file

---
# Security Section - Secure Working Agreement (Humans + Coding Agents)
**Applies to:** every change in this repository (code, docs, CI, infra, data, models, configs)
**Language:** “MUST/SHALL” are mandatory. “SHOULD” is strongly recommended. “MAY” is optional.

## 0) Summary
1) **NO SECRETS — EVER.** No keys/tokens/passwords/cookies in code, logs, screenshots, issues, PRs, fixtures, examples.  
2) **NO SECURITY REGRESSIONS.** If unsure, choose the safer option, reduce scope, add tests, and explain tradeoffs.  
3) **LEAST PRIVILEGE ALWAYS.** Default-deny, minimal scopes, minimal permissions, minimal blast radius.  
4) **ASSUME ALL INPUTS ARE HOSTILE.** Includes user input, files, webhooks, environment vars, DB content, and **LLM outputs**.  
5) **SAFE TOOL USE.** Don’t run destructive commands or risky actions without explicit human approval.  
6) **SUPPLY-CHAIN FIRST.** Pin dependencies, pin CI actions, verify provenance, generate SBOMs, and ship reproducibly.
7) **Follow OWASP TOP 10 Guideline** make sure the project is not vulnerable to any of owasp top 10 security vulnerabilities.

---

## 1) Security invariants (must always hold)
### 1.1 Secrets & credentials (zero tolerance)
- Repos **MUST** remain free of secrets in:
  - source code, configs, docs, examples, tests, fixtures, screenshots
  - CI logs, debug output, stack traces, artifacts
- Secrets **MUST** be injected at runtime via:
  - environment variables, secret managers, CI secrets, workload identity
- Logs/errors **MUST** redact:
  - `Authorization` headers, cookies, bearer tokens, API keys, private keys
- If a secret is exposed: **treat as compromised** and rotate/revoke.

### 1.2 Authentication & session security
- Authentication **MUST** use proven frameworks/middleware.
- Sessions/tokens **MUST** be:
  - short-lived for privileged contexts
  - scoped and audience-restricted
  - stored securely (HttpOnly/Secure/SameSite for cookies)
- CORS **MUST** be explicit allowlist; no `*` for credentialed requests.

### 1.3 Authorization (server-side, default-deny)
- Every privileged action **MUST** enforce server-side authorization.
- Client/UI checks **MUST NOT** be treated as security controls.
- Policies **SHOULD** be centralized (policy middleware, guard layer).
- “Default deny” **MUST** be the baseline.

### 1.4 Input validation & safe parsing
- Validate at **trust boundaries**:
  - API entry points, CLIs, file ingestion, webhooks, websocket
- Use **allowlists** whenever possible:
  - file types, URL schemes, hostnames, IP ranges, commands, enums
- Protect against injection:
  - SQL/NoSQL/GraphQL/prompt injection, command injection, template injection, SSRF

### 1.5 Safe output handling
- Outputs **MUST** be encoded for their context:
  - HTML, JSON, shell, SQL, templates, URLs
- Never build commands/queries by concatenating untrusted input.

### 1.6 Crypto & key management
- Passwords **MUST** be hashed with modern password hashing (never reversible).
- TLS **MUST** validate certificates and hostnames; **MUST NOT** disable verification.

### 1.7 Container & runtime
- Container images **MUST** be minimal and regularly patched.
- Containers **SHOULD** run as non-root; drop capabilities; read-only FS when feasible.

---

## 2) Tool-use policy (command execution)
Default stance: **read-only first**.
- Allowed (safe by default): `git diff`, `git status`, `rg`, `ls`, `cat`, `sed -n`, `python -m pytest` (project tests), linters
- Not allowed without explicit human approval:
  - destructive operations (`rm -rf`, overwrites, chmod -R, migrations on real DB)
  - privilege changes (IAM/policy edits, firewall rules, prod deploys)
  - network scanning/exploit tooling, offensive security operations
  - running arbitrary scripts downloaded from the internet

For any risky operation, the agent **MUST** provide:
- what it plans to do
- why it’s needed
- the safest exact command(s)
- a rollback plan

---

## 3) Definition of Done (security edition)
A change is “done” only if:
- [ ] No secrets added anywhere (including docs/tests/logs/examples)
- [ ] Inputs validated at boundaries; outputs safely encoded
- [ ] Errors/logging do not leak sensitive data
- [ ] Limits/timeouts/rate controls exist where applicable
- [ ] Tests added/updated for security-relevant behavior
- [ ] make the LLM to security check, according to OWASP TOP 10
