# Specialist lens: HTTP, identity, and API

Use when the code parses HTTP at a proxy or gateway, sets caching, or handles sessions, OAuth or OIDC, JWTs, MFA, recovery, or API keys. The access control lens asks whether a principal may act. This one asks whether the HTTP or identity layer can confuse which principal, request, assurance level, or token the action belongs to.

## Discipline

- Framing and cache claims need two components that disagree. Name both and the exact value each one sees.
- For every credential, find the verification and every binding its role needs: issuer, audience, origin, client, session, principal, resource, assurance level, expiry, and one-time use.
- `Host`, `Forwarded`, `X-Forwarded-*`, `Origin`, `Referer`, redirect targets, and callback state are trust decisions. Trace each to the identity or response it affects.
- A missing header, cookie flag, MFA prompt, or rate limit is not a finding alone. Require an accepted invalid request, cross-principal impact, assurance downgrade, or disclosed credential.
- Proxy chains, edge cache keys, identity provider policy, and deployed auth modes may not be in the repo. If one decides the claim, it is `needs_validation`.

## Requests and caches

- **Request framing:** multiple `Content-Length` values, `Transfer-Encoding`, HTTP/2 downgrade, header-name normalization, CR/LF handling. Matters mostly for proxies and custom parsers.
- **Cache poisoning:** a request value changes the response but is absent from the cache key: forwarded host or scheme, cookies, query normalization, language or device headers, auth state.
- **Cache deception:** a private dynamic path treated as a public static asset, through suffix or path-parameter normalization, or a response cached without its identity inputs.
- **Forwarded-header trust:** host or proxy metadata decides absolute URLs, tenant routing, callbacks, reset links, cache keys, or the client IP used for limits and bans. Confirm trusted ingress strips client-supplied copies.
- **Response-header injection:** input reaching `Location`, `Set-Cookie`, or CSP with control characters. Check framework rejection first.

## Browser sessions

- **CSRF:** a cookie-authenticated mutation accepting a cross-site request with no effective token, same-site binding, or strict `Origin` check. Inventory form, JSON-like, multipart, method-override, and legacy routes. State changes over GET count. Check login CSRF separately. Read-only actions and routes needing a bearer token don't qualify.
- **Session fixation and invalidation:** identifiers not rotated on login, account switch, MFA completion, or impersonation; sessions still valid after logout, password change, revocation, or disable. Check signed cookies, refresh tokens, WebSockets, and cached copies.
- **Cookie scope:** an over-broad `Domain` or `Path`, or a sibling cookie another component reads differently. Missing flags alone are hardening unless a less-trusted origin or network position can gain or replace the credential.

## Federated identity

Establish the role first. Redirect allowlisting belongs to the authorization server; token verification belongs to the consumer. One application can be both.

- **JWT:** signature verified with a server-pinned algorithm and key; then `exp`, `nbf`, `aud`, `iss`. Treat `kid`, `jku`, and `x5u` as untrusted. Look for decode-without-verify paths. A valid token for another service is invalid here.
- **OAuth and OIDC:** exact `redirect_uri` ownership; session-bound `state`; PKCE and code binding and one-time use; ID token issuer, audience, signature, nonce; selected provider bound in multi-provider flows. Compare first callback, retry, and account-link routes.
- **SAML:** the element whose signature is checked must be the element used as identity. Check parser safety, validity windows, audience, and replay.

## Assurance and account changes

- **MFA enrollment:** enrolling, replacing, or disabling a factor, generating recovery codes, or trusting a device needs the intended prior assurance. A first factor alone must not replace the second.
- **Step-up:** a challenge must upgrade the right session, account, and action, once. Compare UI, API, batch, and resumed flows.
- **Passkeys:** challenge, RP ID, origin, credential membership, and user verification, bound to the initiating session. Watch for credential-to-account confusion.
- **Account linking:** adding an identity provider, email, or device needs a current session, proof of ownership of the new identity, and callback state bound to the initiating account. Check invite acceptance and unlink/relink for identifier collisions. Linking by an unverified email is account takeover.
- **Recovery:** reset tokens, backup codes, support recovery, and email change are often the weakest door. Check randomness, user and action binding, expiry, one-time use, delivery URL trust, and whether prior tokens and sessions die. For email change, confirm notice to the old address and proof from the new one, with fresh authentication.
- **Password storage:** Argon2id, scrypt, bcrypt, or high-iteration PBKDF2. Fast hashes are a finding when they hold real passwords.
- **Enumeration:** differing login, reset, or signup responses or timing. Only a finding when the existence of the account is itself sensitive or enables a concrete attack.

## API credentials and API design

- **Key scope:** a key reaching broader tenants, resources, or environments than its record grants, or request parameters overriding that. Review lookup, prefix versus full-secret comparison, type confusion between publishable and secret keys, rotation, revocation caches, and bulk endpoints.
- **Key exposure:** keys in URLs, client bundles, redirects, logs, errors, or build output. A public identifier called a key is not a secret. Confirm the type and the authority gained.
- **Service identity headers:** a backend trusting identity headers from any peer. Only the trusted proxy may set them, and it must strip incoming copies.
- **Object, property, and function authorization:** check each separately. Responses that serialize whole records leak fields; updates that spread a request body allow mass assignment. Allowlist in both directions.
- **Request limits:** body size, content type, batch size, query depth or cost.

## Moves

- Walk issue → store → transmit → consume → refresh → revoke for every credential, across normal, error, retry, migration, and account-switch paths.
- List every door to the same identity. The real policy is the weakest parallel path.
- For recovery and linking, draw the account graph before and after. Each new edge needs a principal, proof, assurance level, binding, and revocation effect.
