# Specialist lens: cloud, edge, build, and release

Use when the repo defines deployment, edge workers, cloud identity, containers, ingress, environment configuration, or the path from source to a shipped package. For GitHub Actions use `lenses-ci.md`.

## Deployment discipline

- Don't infer live exposure from a manifest alone. Establish which environment uses it, what overrides it, and whether the path is active.
- Broad policy is a finding only when lower-trust input can reach an unauthorized action.
- Secret references aren't disclosure. Require a lower-trust reader, output, or unsafe fallback.
- Account policy, network attachment, and drift need the owner to observe them: `needs_validation`.

## Identity and reachability

- **Workload overreach:** a worker, function, or VM identity reaching tenants or resources beyond its role, with request input choosing the target. Worker bindings count as identity: which worker holds which secret, service binding, or database credential?
- **Metadata as authority:** an app trusting caller-supplied identity headers, tags, or account IDs without proof they came from the platform or trusted proxy.
- **Unexpected reachability:** an admin, debug, metrics, or internal route public by default, a catch-all route, or a service binding also exposed on a public hostname. A repo-controlled public route to a sensitive handler can be `confirmed`.
- **Trusted-proxy bypass:** a backend accepting forwarded identity from peers outside the intended path, or failing open when the proxy is absent.
- **Metadata services:** an untrusted URL reaching instance metadata or internal APIs with workload credentials. Applies to any VM in the estate, not to Workers.

## Configuration and secrets

- **Precedence drift:** dev values, defaults, environment variables, flags, or per-environment overrides disabling auth, transport security, or tenant scope in production. Render the final config for each environment, not just the base file.
- **Secret spread:** secrets in logs, crash reports, process arguments, build output, or read APIs reachable by another workload.
- **Renewal fallback:** a failed credential refresh leaving a stale credential active, or the app accepting a weaker identity mode.
- **Edge and origin mismatch:** an edge runtime assuming a secret, API, or tenant policy that differs from the origin, so falling back to origin changes authority or caching.
- **Event source identity:** a worker trusting event body fields instead of a signed envelope. Compare push, retry, and dead-letter paths.
- **Containers, if any:** running as root, secrets in `ENV`, `ARG`, or copied files, `--privileged`, a mounted Docker socket, missing `.dockerignore`. These matter through attainable impact.

## Supply chain discipline

- A mutable or known-vulnerable dependency isn't a finding alone. Show who influences resolution, which build consumes it, and what boundary follows.
- Follow integrity across each handoff: source, resolved inputs, build, artifact, test result, signature, promotion, update consumer.
- A checksum fetched from the same untrusted place as the artifact proves nothing.

## Inputs and build

- **Namespace confusion:** resolver config choosing an unintended registry, or an internal package name that could be claimed publicly. Check scopes, `.npmrc`, and fallback order.
- **Typosquats:** dependency names one character from a popular package, or look-alike Unicode.
- **Mutable inputs:** branches, tags, downloaded tools, remote includes, or floating container tags reaching trusted output.
- **Generated content:** schemas, vendored archives, generated clients, or binary blobs shipped without the review source gets.
- **Build context:** secrets, local config, or fixtures entering a package or image because ignore rules are too wide. Check the `files` field and `.npmignore` for published packages. Confirm a real credential is exposed.
- **Install scripts:** `preinstall` and `postinstall` in the repo's packages and dependencies. Look for network calls, obfuscation, and environment reads.

## Release

- **Build-to-promotion swap:** tests, review, and publication referring to mutable names rather than the same immutable digest.
- **Release authorization:** a release accepted from the wrong workflow, branch, or key role. Check that consumers validate identity, not just the presence of a signature.
- **Update trust:** an updater authenticating bytes but not version, channel, or rollback state. Includes `npx`-based install flows and anything the CLI downloads.
- **Plugin trust:** extensions gaining more host authority than declared, or install hooks running before checks. A user installing their own plugin for themselves isn't a boundary.

## Moves

Walk back from a released package to every source, generated input, credential, cache, test, and authorization decision. Check the failure paths: revoked key, failed download, partial release, rollback, registry outage. Make a table per environment: exposed route, workload identity, peers, bound secrets, and resources.
