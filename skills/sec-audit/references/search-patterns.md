# Search patterns

Starting points for `rg`. A hit is a place to read, never a finding. Pipe each search through the mask from `SKILL.md` rule 4, because a hit can be a line holding a credential: `<search> | rg --passthru -r '<redacted>' "$SECRET_RE"`. Add `-g '!**/node_modules/**' -g '!**/*.test.*'` when you want source only, and read test hits separately only if they run in CI with credentials.

## TypeScript and JavaScript

```sh
# Code execution
rg -n 'eval\(|new Function\(|vm\.run|setTimeout\(\s*["`]|setInterval\(\s*["`]'
rg -n 'child_process|execSync\(|exec\(|execFile\(|spawn\(|fork\(|shell:\s*true'

# HTML and DOM sinks
rg -n 'dangerouslySetInnerHTML|\.innerHTML|\.outerHTML|insertAdjacentHTML|document\.write|bypassSecurityTrust|v-html'
rg -n 'href=\{|src=\{|window\.open\(|location(\.href)?\s*='

# SQL and query builders
rg -n '`\s*(SELECT|INSERT|UPDATE|DELETE)[^`]*\$\{|\.(raw|unsafe|query)\(\s*`|\.rpc\('
rg -n 'format\(|EXECUTE\s' -g '*.sql'

# Outbound requests with variable targets
rg -n 'fetch\(\s*[a-zA-Z_`]|axios\.|got\(|http\.get\('

# Prototype pollution and merges
rg -n '__proto__|constructor\[|Object\.assign\(|\b_\.merge\(|deepMerge'

# Crypto and randomness
rg -n 'Math\.random\(|createHash\(.(md5|sha1)|===\s*(secret|token|signature|hmac)|(secret|token|signature)\s*==='

# Trust in client values
rg -n 'x-forwarded|cf-connecting-ip|x-real-ip|req\.headers\.host|headers\.get\(.host'
rg -n 'NEXT_PUBLIC_' -g '!*.md'

# Fail-open shapes
rg -n 'catch\s*(\([^)]*\))?\s*\{\s*(return true|return|\})'
rg -n 'TODO.*(auth|secur|valid)|FIXME.*(auth|secur|valid)|HACK|skip.*(auth|check)'
```

Type annotations prove nothing at runtime: `req.body as UserInput` and `any` are unvalidated. Look for a runtime schema such as `zod.parse` on the path.

Safe, don't flag: JSX `{value}`, `textContent`, `createTextNode`, parameterized `$1` queries, ORM filter objects, and `res.json`.

## Sources worth tracing

Request `query`, `body`, `params`, `cookies`, and most `headers`; `formData()`; `searchParams`; `location.*`; `document.referrer`; `postMessage` data; webhook payloads before verification; inbound email and Telegram messages; model output and tool arguments; `process.argv` and project config in CLIs.

## SSRF bypass forms

When a URL check exists, check it survives: encoded dots; octal, hex, and decimal IPs; `127.1`; any `127.x`; `[::1]`, `[::ffff:127.0.0.1]`; redirects to internal hosts; DNS rebinding between check and use; `0.0.0.0`; link-local `169.254.0.0/16`; userinfo tricks like `https://good@evil`; and scheme confusion.

## Python

```sh
rg -n 'pickle\.loads?|yaml\.load\(|marshal\.loads?|shelve\.open' -g '*.py'
rg -n 'os\.system|os\.popen|shell=True|eval\(|exec\(' -g '*.py'
rg -n 'execute\(\s*f["'"'"']|execute\([^)]*%|text\(\s*f["'"'"']|render_template_string' -g '*.py'
```

`yaml.safe_load` and parameterized queries are safe. `random` is fine except for security tokens, which need `secrets`.

## GitHub Actions

```sh
rg -n 'pull_request_target|workflow_run|issue_comment' .github/workflows
rg -n '\$\{\{[^}]*(github\.event\.|github\.head_ref)' .github/workflows
rg -n 'uses:' .github/workflows | rg -v '@[0-9a-f]{40}|uses: \./|uses: (actions|github)/'
rg -n 'write-all|id-token: write|self-hosted|upload-artifact|download-artifact|actions/cache' .github/workflows
```

## Containers

```sh
rg -n '^USER|^ENV .*(PASS|SECRET|KEY|TOKEN)|^ARG .*(PASS|SECRET|KEY)|:latest|^ADD |docker\.sock|--privileged'
```

## Notes on scanner templates

Semgrep and CodeQL aren't installed here, so these rules weren't tested against them. If they're installed later, build rules from the source and sink lists above. Trail of Bits' JavaScript templates are a starting point only: their sanitizer list is not sink-specific (`encodeURIComponent` clears a shell sink), they miss `execFile`, `fork`, `dangerouslySetInnerHTML`, and header and `postMessage` sources, and they only exclude `.js` tests.
