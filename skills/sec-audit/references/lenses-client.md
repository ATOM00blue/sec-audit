# Specialist lens: client side and browser

Use when trust decisions or untrusted rendering happen in a browser: single-page apps, service workers, code rendering attacker content into the DOM, cross-window messaging, or browser storage. These include sources the server never sees, such as URL fragments, `window.name`, and `postMessage`.

## Discipline

- A candidate needs a controllable source and an executing or disclosing sink. Name both and show the path.
- Impact must reach a victim's session, another origin, or shared persistence. Injecting into your own page or seeing your own data is not a finding.
- Framework escaping, same-origin policy, CSP, COOP/CORP, and modern `noopener` defaults are real controls. Check them before assigning impact. Deployed headers outside the repo mean `needs_validation`.

## DOM and object state

- **DOM XSS:** `location`, `document.referrer`, `window.name`, message data, and storage flowing to `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`, string-evaluating APIs, executable URLs, or framework escape hatches.
- **URL attributes:** `href` and `src` with user values need a scheme check, because `javascript:` and `data:` run. Framework escaping doesn't cover this.
- **Sanitizers:** `dangerouslySetInnerHTML` with DOMPurify on output is the safe form. Check configuration and that nothing mutates the result afterwards. Markdown renderers that allow HTML count as HTML sinks, including when rendering model output.
- **DOM clobbering:** injected `id` or `name` attributes shadowing a global or config object. Needs both surviving markup and a security-relevant use.
- **Prototype pollution:** an attacker key reaching a recursive merge or path assignment, plus a reachable gadget using the polluted property. `JSON.parse` alone, a shallow copy, or pollution with no gadget isn't enough.

## Messaging and cross-origin

- **`postMessage`:** a sensitive handler with no exact origin allowlist, or sensitive data sent to `*`. Substring, prefix, and unanchored regex origin checks don't count.
- **Cross-site WebSocket:** an upgrade accepting ambient cookies from any origin, with no `Origin` check or channel token, and a handler that matters.
- **Credentialed CORS:** a reflected or weakly matched `Origin` with credentials allowed and a sensitive response. A wildcard with credentials is rejected by browsers, so report the real allowed path.
- **Client-side CSRF:** a URL fragment or parameter choosing the target of a same-origin state-changing request.

## Service workers and storage

- **Worker takeover:** attacker content becoming the worker script, an over-broad scope, or update imports without integrity.
- **Worker cache confusion:** personalized responses cached without account or auth state, and served after logout or account switch.
- **Storage:** tokens or private data readable by a less-trusted same-origin component or a later account, or still used after logout. Storing a token alone isn't a finding.
- **Cross-tab:** `BroadcastChannel`, `storage` events, or shared workers carrying identity between tabs without a session binding.

## Leaks and UI

- **XS-Leaks:** a page distinguishing protected state through load events, frames, redirects, timing, or size. Needs one concrete secret-bearing fact.
- **Clickjacking:** a framed, state-changing action with no `frame-ancestors` or equivalent. Read-only pages are hardening notes.
- **Navigation:** redirects controlled by a client source without a scheme and destination policy. Open redirects count when they feed OAuth flows or phishing with a concrete result.
- **Secrets in bundles:** `NEXT_PUBLIC_*` values and anything imported into client components.

## Moves

Start from sinks and trace back to browser-only and server sources. Consider account switch, logout, worker update, offline fallback, and stale tabs.
