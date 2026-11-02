# Specialist lens: GitHub Actions

Use for `.github/workflows/*`, `action.yml`, and local actions under `.github/actions/`, plus anything those run: scripts, `Makefile`, and agent instruction files such as `CLAUDE.md` and `AGENTS.md`.

## Threat model

Say who the attacker is. On a public repo, anyone can open fork PRs, issues, and comments. On a private repo, name the realistic source instead: a compromised dependency or third-party action, a low-privilege collaborator, or text from outside that reaches a workflow.

Don't flag what already requires write access to main: `workflow_dispatch` inputs, `schedule`-only jobs, expressions in `push` workflows on protected branches, or `workflow_call` inputs with only internal callers. Branch protection is a deployed fact; if it decides the claim, it is `needs_validation`.

## What a finding needs

Entry point, payload, how the payload runs, what the attacker gains, and steps an attacker would follow. Missing one makes it `needs_validation` or nothing. Read the full YAML rather than trusting grep, including `if:` conditions and existing mitigations.

## Checks by trigger

**`pull_request_target` with fork code.** Runs with the target repo's secrets. Dangerous when it checks out the PR head or merge ref and then runs anything from it: install scripts, tests, build files, `go run`, `Makefile`, shell scripts, or local actions. Without such a checkout it is usually safe, but check for consumed artifacts or commands.

**Expression injection.** `${{ }}` in a `run:` block is substituted before the shell parses it. Attacker-controlled values include:
- PR title, body, head ref, and head label; `github.head_ref`
- issue title and body; comment and review body; discussion title and body
- wiki page names; filenames from a changed-files step

Branch names and filenames can carry `$()`, `${IFS}`, and base64 payloads.

Not injectable: PR number, `github.sha`, run IDs, repository name, `secrets.*` as a value. Expressions in `if:` and `with:`, and those assigned in `env:`, aren't shell-expanded. The fix is to pass the value through `env:` and use it as a quoted shell variable.

Check downstream too: a value assigned safely to `env` becomes unsafe if an `eval` or an action's own script expands it. A `with:` value can also reach a composite action's `run:`.

**Comment commands.** `issue_comment` handlers need an `author_association` check for `OWNER`, `MEMBER`, or `COLLABORATOR`. Without it, any commenter runs the command. Worse if it also checks out PR code or expands the comment body.

**`workflow_run` and artifacts.** Artifacts aren't signed. A privileged workflow that downloads an artifact from an untrusted run and executes or deploys it is running attacker output, as is one reading a value from it into a shell.

**Caches.** A lower-trust job populating a cache a release job later restores. Check key predictability and scope.

**Self-hosted runners.** They persist between jobs. Untrusted triggers on them allow credential theft, backdoors, and network pivot. Check runner group isolation.

## Credentials and permissions

- Rank each secret by blast radius. A classic PAT reaches every repo its owner can. npm tokens reach downstream users.
- A missing `permissions:` block inherits the repo default; a deployed fact. `write-all` or `contents: write` on a PR trigger only matters with a path for untrusted code.
- Secrets at workflow or job level are visible to every step, so any injection can read them.
- Masking covers only the literal value. Base64 or truncated forms print in logs. Secrets written to files can leave through artifacts when `path: .`.
- OIDC: a cloud trust policy with `repo:org/repo:*` lets any ref assume the role. Prefer an environment-bound subject.
- Environments give required reviewers and branch limits, but only if configured; a referenced missing environment is created unprotected.

## Third-party actions

Pin third-party actions and reusable workflows to a full SHA, and report only when the job has secrets, OIDC, write scope, release, deploy, or publish power. Read-only unprivileged CI is not a finding. Version tags on `actions/*` and `github/*`, and same-repo local actions, are not pinning findings. A local action can still be a problem through a fork checkout. A SHA-pinned action that downloads and runs a remote script at runtime remains mutable.

## AI agents in CI

An agent running on PR content reads attacker-editable `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, and similar as trusted instructions. Dangerous with `pull_request_target`, write permissions, broad tool access, or `allowed_non_write_users: '*'`. The agent noticing the attack is not a control. Fix the trigger, narrow the tools and token, and cover the instruction files with CODEOWNERS.

## Report additions

Add a "Reviewed and cleared" list of workflows checked and found safe, with the reason. If there are no findings, say so. Don't invent any.
