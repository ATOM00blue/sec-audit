# sec-audit

An end-to-end security skill for coding agents. It works in Claude Code and Codex.

It reviews a single change or a whole codebase, along with secrets, dependencies, CI/CD, deployment, integrations, and agent skills and hooks. Every finding is traced from entry point to impact, then checked by a separate agent that tries to disprove it.

## Install

For Claude Code and Codex, in every project:

```sh
git clone --depth 1 https://github.com/ATOM00blue/sec-audit ~/.sec-audit && mkdir -p ~/.claude/skills ~/.agents/skills && ln -s ~/.sec-audit/skills/sec-audit ~/.claude/skills/sec-audit && ln -s ~/.sec-audit/skills/sec-audit ~/.agents/skills/sec-audit
```

Start a new session afterwards, because a running session may not pick up the skill or later changes to it. To update, run `git -C ~/.sec-audit pull`.

## Use

In Claude Code, type `/sec-audit`. In Codex, type `$sec-audit`, and start Codex with `codex -s read-only` unless the mode writes a report.

| Command | What it does |
|---|---|
| `/sec-audit` | Reviews the current branch's pending changes |
| `/sec-audit diff [base]` | Reviews the change against a base branch |
| `/sec-audit full [quick\|standard\|deep] [paths]` | Audits the codebase or the named paths |
| `/sec-audit infra [secrets\|deps\|ci\|deploy\|integrations\|skills]` | Audits infrastructure, or one area of it |
| `/sec-audit variants <finding>` | Finds other instances of a known bug |
| `/sec-audit recheck <finding>` | Checks from fresh evidence whether a finding is really fixed |
| `/sec-audit doctor` | Checks the setup and which scanners are installed |

`full` and `infra` write their reports to `../security-audits/`, outside the repo. Create that folder before running them.

## What it checks

- **Application:** injection, access control, authentication, sessions, business logic, race conditions, file handling, outbound requests, and cryptography.
- **Data and clients:** multi-tenant isolation, browser security, API design, webhooks, and protocols.
- **AI features:** prompt injection that crosses a real boundary, and agent tools.
- **Infrastructure:** committed secrets, dependencies, GitHub Actions, cloud and deployment config, and release paths.

## How it works

1. **Model:** maps the attackers, assets, entry points, and boundaries.
2. **Plan:** picks the relevant checks and, for audits, tracks coverage unit by unit.
3. **Hunt:** agents trace each input from entry point to sink.
4. **Critic:** on audits, a fresh agent looks for what was missed.
5. **Verify:** a separate agent tries to refute each finding without seeing the hunter's reasoning.
6. **Variants:** searches for other instances of each confirmed bug.
7. **Report:** gives each finding a verdict, severity, confidence, and evidence, and says plainly what wasn't assessed.

## Safety

- It's read-only. It doesn't change code, run the project, or contact live services.
- Credentials are masked in diffs and searches, and reports give only a credential's type and location. The mask recognizes known formats, so an unusual credential can still be printed.
- Code comments and docs are treated as evidence, never as instructions.

## Project profile

To give the skill your threat model, add `.sec-audit/profile.md` to your repo. [The template](skills/sec-audit/references/profile-template.md) shows what it can hold.

## Limits

- No tool can promise to find every vulnerability. A report covers what was assessed and names what wasn't, and it never declares code secure. This doesn't replace a professional audit.
- Findings come from reading the source, so they aren't proof of a working exploit.
- `diff` and `doctor` have been tested, with `diff` run against a repo of planted bugs. `full`, `infra`, `variants`, and `recheck` are new and haven't been run end to end.
- A small `diff` used about 250,000 tokens in testing. Audits use far more.

## License

[CC BY-SA 4.0](LICENSE). See [NOTICE.md](skills/sec-audit/NOTICE.md) for third-party credits and licenses.
