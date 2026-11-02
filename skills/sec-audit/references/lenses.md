# Hunting lenses

Paste the full text of each chosen lens into the hunter's prompt. Choose a lens because a boundary exists in the code, and record why the others were left out. These are questions to investigate, not findings. A hit becomes a candidate only when it meets the six-part claim in `SKILL.md`.

This file holds the general lenses. Read a specialist file only when its boundary is in scope, and paste the relevant sections, including its discipline section:

| File | When |
|---|---|
| `lenses-web-auth.md` | HTTP parsing, caching, sessions, OAuth, JWT, MFA, recovery, API keys, API design |
| `lenses-resources.md` | Untrusted work can consume shared capacity or paid APIs, or crash a shared service |
| `lenses-data.md` | Multi-tenant storage, derived copies, exports, migrations, deletion, retention |
| `lenses-client.md` | Browser rendering, messaging, storage, service workers |
| `lenses-cloud-release.md` | Deployment config, edge workers, cloud identity, dependency resolution, publishing |
| `lenses-ci.md` | GitHub Actions workflows and local actions |
| `lenses-protocols.md` | Webhooks, queues, streaming, RPC, internal messages |
| `lenses-local.md` | CLIs, local daemons, native or `unsafe` code |

`search-patterns.md` has `rg` starting points for hunters.

## Code lenses

### Injection and unsafe sinks
Trace input to SQL (including string-built queries and raw RPC or PostgREST filters), shell commands, templates, file paths, redirects, HTML, and deserialization. Check indirect paths: data stored safely, then used in a dangerous context by other code. Check injection through keys, field names, headers, and metadata, not just values, and into secondary systems such as search, analytics, and caches. For stored procedures, read the procedure: dynamic SQL inside a `SECURITY DEFINER` function is a sink.

### Access control
Don't stop at "a check exists". Is it the right check, on the right resource, by the right mechanism?
- Do endpoints authenticate but forget to authorize the specific object or tenant?
- Is there another path to the same state change with a weaker check?
- Can a request body field override what permissions restrict (mass assignment, role, owner, plan, or balance fields)?
- Do batch, bulk, export, and import check every item?
- Can both sides of an ownership comparison be null or empty at once?
- Do `SECURITY DEFINER` functions, service-role clients, and row-level policies re-check the caller?
Use two-user and two-tenant reasoning when describing the result.

### Authentication and sessions
Follow session creation, refresh, logout, and revocation. Does a terminated session or revoked key stop working everywhere, including caches? JWT: algorithm, audience, issuer, expiry. OAuth and OIDC: exact redirect match, state, PKCE, client binding, code reuse, account linking by unverified email. Check recovery, invite, and email-change flows for takeover. API keys: storage, comparison, scope, prefix confusion, legacy formats. MFA: can a lower assurance session reach a step-up action? Check enumeration through responses and timing where it matters.

### Files, outbound requests, and parsers
Path traversal through encoding, symlinks, and archive extraction. Upload names, types, and size. SSRF: user-influenced host, protocol, or path, followed through redirects, DNS rebinding, and URL parser differences, including webhook and callback URLs and link previews. Unsafe deserialization and XML parsers. TOCTOU between check and use.

### Cryptography and secrets in code
Weak randomness for tokens, keys, or nonces (`Math.random`, predictable IDs). Non-constant-time secret comparison where timing is reachable. Missing HMAC verification, nonce reuse, unauthenticated encryption, static IVs, and key reuse across purposes. What happens when a crypto call fails: does it fall back to no verification? Hashes used for checksums and non-security randomness are fine.

### Business logic and money
These flaws don't look like bugs to a scanner. For each major workflow:
- **State:** skip a step, go backwards, replay a completed flow; if step 2 of 3 fails, is step 1 rolled back?
- **Races:** check-then-act without atomicity, such as double spend, double grant, lost updates, and one-per-user limits enforced in memory or per instance.
- **Numbers:** negative, zero, overflow, rounding, floats for money, unit mismatches, string and number coercion.
- **Trust:** prices, plans, or limits taken from the client; values assumed safe because another path validated them.
- **Time:** expiry at the exact boundary, clock source, window edges, timezone differences.
- **Idempotency:** retried or replayed events that credit twice; refunds and chargebacks that leave value behind.
- **Defaults:** the outcome when config is missing, a flag is off, a dependency is down, or a migration is half applied.

### Feature abuse and data leakage
Legitimate features used for other purposes. Exports and backups that include data above the caller's access, or deleted data. Imports that bypass validation or overwrite others' records. Search, filter, and sort as an oracle for hidden values. Enumeration from differing errors, sizes, or timing. Preview or draft tokens that unlock more than one item. Cache headers or cache keys that let a shared layer serve private content.

### Chains and second-order use
Compare what component A guarantees with what component B assumes: truncation, coercion, normalization, tenant scope. Stored strings that later become URLs, paths, templates, regexes, or policy expressions. Capabilities that grow through delegation, refresh, caching, role change, or composition. Windows around soft delete, revocation, cache expiry, and migration. Restore and rollback paths that skip current ownership checks. Confirm each link in a chain; never assume a downstream effect.

### Client side
Escape hatches with attacker data: `dangerouslySetInnerHTML`, `innerHTML`, Markdown renderers that allow HTML, `javascript:` URLs. Open redirects from `next`, `return`, or `url` parameters. `postMessage` without an origin check. CORS that reflects origins with credentials. CSRF on cookie-authenticated state changes. Cookie attributes on session cookies. Secrets shipped in client bundles (`NEXT_PUBLIC_*` and similar). Service worker and browser storage holding credentials. Prototype pollution through object merges only where a real gadget exists.

### AI, LLM, and agent features
Draw four maps: execution identities, capabilities, writable context or memory sources, and output destinations. Then connect the principal at the start to the authority at the end. Start from side-effecting tools and work back through dispatcher, schema, confirmation, context, and ingestion.
- Who can write what enters another principal's context: tickets, emails, documents, tool results, memory?
- Are prompt caches, conversation stores, embeddings, and retrieval keyed by tenant in the query itself?
- Do tool handlers re-check the requester's authority, or run under a service identity?
- Does an approval bind the exact tool, arguments, target, and amount, or can a later turn change them?
- Do model-produced arguments reach SQL, shell, file, URL, or git sinks unchecked?
- Does a schema accept aliases, extra fields, or coercions the handler reads differently?
- Is model output rendered as HTML or Markdown that loads remote resources, or run as commands?
- Can a bounded request trigger unbounded paid work, sends, or mutations?
- MCP: audience-bound tokens, no token passthrough, tool identity tied to the authenticated connection, metadata never treated as policy.
Prompt text alone is not a finding. Require deterministic code that grants or fails to limit authority.

### Resources and cost
Work that untrusted input can multiply: unbounded body or parameter size, expensive regex or parsing on shared workers, unbounded fan-out, queues that accept more than workers can drain, retries that amplify. Who pays? Check that spend limits are enforced before the expensive call, that they count what is actually consumed, and that they hold across instances, concurrent requests, and failed attempts. Limits kept in per-instance memory don't hold globally. Reason from the code; never exercise it.

### Data lifecycle
Classify what is stored: credentials, personal and payment data, prompts and responses, internal metadata. Trace collection, logs, analytics, third-party transfer, retention, export, backup, and deletion across tenants. Does deletion reach derived copies, caches, and search? Does restoring from backup or a migration lift a past deletion or restriction? Do zero-retention promises hold on error paths and in logs? Report observed protection and uncertainty; don't draw legal compliance conclusions.

### Exceptional conditions
Fail-open paths: `catch` blocks that allow, a default of `true`, a timeout treated as success, an auth or billing dependency being down. Partial state after a thrown error. Error responses that return stack traces, internal paths, or SQL. Cleanup and rollback that don't run. Inverted conditions and functions whose return value contradicts their name.

### Basics (thorough and literal)
Check each item. Then trace its impact before calling it anything.
- Hardcoded credentials and private keys; tracked `.env`, `*.pem`, and `*.key` files; `.gitignore` coverage.
- Comments such as `TODO: auth`, `FIXME: validate`, or `HACK: skip`.
- Debug modes that a header, query parameter, or environment value can enable.
- Unprotected `/debug`, `/admin`, `/test`, `/metrics`, `/env`, `/config`, and internal routes.
- Seed or test credentials that could work in production.
- Dynamic execution with input: `eval`, `new Function`, `exec`, `execSync`, `child_process`, dynamic `import()`, `pickle.loads`, `yaml.load`, `os.system`, `subprocess` with `shell=True`.
- CORS wildcards with credentials; session cookies without `HttpOnly`, `Secure`, or `SameSite`.
- Password hashing with fast hashes; security tokens from non-crypto randomness.

### Wildcard
No assigned category. Read what looks boring, half finished, experimental, or bolted on. Which API calls does the client never make but the server accepts? What happens when features that weren't designed together are mixed? What do the tests avoid? Are there reverted security fixes or removed auth checks in git history? Which comments explain why something is safe, and are they right? Anything named `temp`, `hack`, or `legacy` gets read closely. The evidence bar is the same as for every other lens.

## Infrastructure lenses (`infra` mode, and `full` when in scope)

### Secret archaeology
Follow rule 4 in `SKILL.md`: no values in output. Search current source and all history with its masking pattern, then by hand for JWTs (`eyJ`), URLs with embedded credentials, inline CI secrets, logged tokens, and agent configuration. Separate placeholders from anything that could confer authority. Don't dismiss a secret because it is old or said to be rotated; ask for proof of revocation. Never test a key. The recommendation is to revoke, rotate, and investigate use; history cleanup is a separate maintenance step.

### Dependency supply chain
Read manifests, lockfiles, and workspace boundaries as data. Never run an install or audit command that executes project config. For each candidate, record affected version evidence, direct or transitive, production and build exposure, reachability of the vulnerable function, known exploitation, and available fix. Look for missing or inconsistent lockfiles, lifecycle scripts, unpinned sources, dependency confusion between public and private names, and packages published from this repo. A low CVSS or a dev classification sets no ceiling; an old package alone proves nothing.

### CI/CD
Trace event → attacker-controlled value, artifact, or cache → execution → credential or write capability. Review `pull_request_target`, `workflow_run`, reusable workflows, `${{ }}` interpolation in `run:`, fork permissions, artifact and cache trust, self-hosted runners, effective `permissions:`, environment protection, and which jobs hold deploy or publish credentials. Unpinned actions and a missing CODEOWNERS are leads, not automatic findings. Pinning limits replacement risk but doesn't make the code trustworthy.

### Infrastructure and environment trust
Read deployment config, IaC, containers, and edge routes as data. What is publicly reachable, and what identity does it run as? Can preview builds, staging, local tools, or maintenance jobs reach production credentials or data? Check database network exposure, service-role use, internal-only routes that rely on an unverified header, and secrets bound to more workers than need them. A root container or wildcard permission matters through the impact it allows. Name every assumption about deployed state; those become owner checks.

### Webhooks and integrations
Trace the full chain before claiming a check is missing. Verify signature over the raw body, timestamp and replay window, idempotency, tenant binding, and whether a forged or replayed event can change money, ownership, or access. Check inbound email and chat bots the same way. OAuth client, audience, and redirect bindings; token scope; TLS verification; outbound redirects. Private networking is a control to verify, not a severity ceiling.

### Skills, hooks, and agent configuration
Treat repo-local skills, plugins, hooks, MCP configs, and setup scripts as code. They can direct what an agent runs. Trace install and update paths through network requests, credential access, shell execution, and external writes. A `curl` command or a familiar publisher is not a verdict either way; check destination control, interpolation, inherited environment, and pinning. Include this skill in scope. User-level and global settings are out of scope unless the user adds them.
