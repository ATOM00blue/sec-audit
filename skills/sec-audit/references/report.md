# Reports and run files

Write reports only after verification. Prose never changes a verdict, severity, or blocker from the verified record.

## Chat report (`diff`, `recheck`)

```markdown
**Status: complete | partial | not assessed**
Scope: <base..head, files reviewed, what was left out>

| ID | Severity | Confidence | Evidence | Location | Result |
|---|---|---|---|---|---|

### F-1: <title> (<severity>)
- **Location:** `path:line`
- **Attacker:** <who, and what they can already do>
- **Path:** <entry point → sink>
- **Code:** <quoted exact code, credential values replaced by `<redacted>`>
- **Result:** <what happens, to whom>
- **Conditions:** <what must be true, including assumed wiring>
- **Controls examined:** <what was checked and why it doesn't stop this>
- **Fix:** <smallest change>, test: <regression test>

### Needs validation
- **V-1: <title>:** `path:line`. If true: <attacker and result>. Blockers: <each exact fact>. Check: <safe way to get them>. Fix if true: <smallest change>.

### Retracted alerts
<any alert already sent during the run that a verifier then rejected: what it was, and why it was rejected>

### Hardening notes
<only concrete items, kept short>

### Follow-up
<out-of-scope variants as unverified leads, and areas not assessed>
```

Status for these modes: **complete** means every file in scope was assessed and every in-scope candidate has a verifier decision, even if gaps in surrounding context are named in the scope. **partial** means some of that didn't happen. **not assessed** means nothing did.

A report that isn't complete opens, before any finding, with `Ran:`, `Missing:`, `Reason:`, and `Next:` lines, so it can't read as a clean result.

Omit sections with nothing in them, including an empty findings table. If there are no `confirmed` and no `needs_validation` records, the report is the status line, the scope, "No supported findings in the assessed scope.", and any hardening notes and Follow-up. Keep leads and alerts in that case; a clean result on the change must not hide them. The `variants` report uses the format in `variants.md`, with the same status line.

## Run directory (`full`, `infra`)

`<repo-root>/../security-audits/<YYYY-MM-DD>-<mode>/` contains:

| File | Contents |
|---|---|
| `coverage.md` | The ledger, updated after every wave |
| `findings.json` | One record per fingerprint, all verdicts |
| `REPORT.md` | The report below |

Check `findings.json` parses with `jq empty findings.json` before writing the report. The parent is the only writer. Never store a secret value in any of these files.

### `coverage.md`

```markdown
Source ref: <sha>, tree clean | dirty. Profile: <profile>. Prior runs read: <paths or none>.

| Unit | Surface | Boundary | Lens | Paths reviewed | Status | Agent | Result |
```

Statuses:

| Status | Requires |
|---|---|
| `planned` | nothing yet |
| `covered` | reviewed paths and invariants checked |
| `candidate` | reviewed paths and at least one linked fingerprint |
| `blocked` | partial evidence and the exact blocker |
| `deferred` | the reason, usually budget or profile |
| `out_of_scope` | the reason |
| `not_applicable` | the source fact that excludes it |

A unit with no reviewed paths can't be `covered`. Only `candidate` units carry fingerprints. Each reviewed path must be tied to an invariant someone checked, and the agent named. When a critic reopens a unit, keep the earlier attempt and its reason, and give the new attempt a fresh agent with no reused evidence. Keep unit IDs stable across runs by deriving them from source objects: `<path-or-route>::<boundary-symbol>::<lens>`. Don't include line numbers, agents, or verdicts. One semantic tuple gets one ID.

Always include these areas as units, so a skipped one shows up as a gap: application model, attack surface, and the lens areas in scope. For `infra`, those are secrets, dependencies, CI/CD, infrastructure, integrations, and skills. For `full`, add OWASP Top 10, STRIDE, data classification, and AI features when present.

### `findings.json`

A JSON array sorted by fingerprint:

```json
{
  "fingerprint": "access-control/apps/web/app/api/thing/route.ts/missing-owner-check",
  "verdict": "confirmed | needs_validation | rejected",
  "title": "...",
  "root_cause": "...",
  "trace": [{"role": "entrypoint|propagation|sink|artifact", "location": "path:line", "quote": "..."}],
  "conditions": ["..."],
  "result": "...",
  "evidence": "traced | reproduced",
  "severity": "critical | high | medium | low | null",
  "confidence": "high | medium | null",
  "confidence_reason": "...",
  "fix": "...",
  "regression_test": "...",
  "challenge": "independent | sequential challenge; independent agent unavailable",
  "dependency": {"affected_version": "...", "reachability": "reachable | unreachable | unknown", "exposure": "production | build | dev", "exploitation": "known | none known | unknown"},
  "variants": [{"location": "path:line", "verdict": "...", "severity": "..."}],
  "blockers": [],
  "owner_check": null,
  "reason_rejected": null,
  "dissent": null,
  "units": ["unit id"]
}
```

Keep `rejected` records so future runs don't repeat a disproved claim. Keep the fingerprint stable for the same root cause across every state and run. It comes from root cause, path or symbol, and advisory IDs, never from title, line, severity, or agent. `dependency` appears only for dependency findings.

A record is malformed, and must be fixed before reporting, if:

- Fingerprints aren't unique and sorted, or any path is absolute or contains `..`.
- A required text field is empty or whitespace.
- It has no trace, or a multi-step trace doesn't start at `entrypoint` and end at `sink`. A credential or configuration finding uses one `artifact` step.
- `confirmed` lacks severity, evidence, or a fix, or has `blockers` or `reason_rejected`.
- `needs_validation` has a severity, or no blockers and no owner check.
- `rejected` has a severity or blockers, or no reason. It keeps the refuted claim on record.
- Severity exceeds the demonstrated result.

These checks prove format only, not truth.

### `REPORT.md`

1. **Status line:** complete, partial, or not assessed, plus the reason if not complete. `quick`, scoped, budget-limited, or interrupted runs are `partial`. A `standard` or `deep` run is `complete` only when no units stay planned, the final critic added nothing, and every candidate has a verifier decision.
2. **Run facts:** mode, profile, scope, source ref, agents planned and spent, scanners used and missing, prior runs used, and a statement that nothing was executed or probed.
3. **Summary:** one short paragraph on security posture.
4. **Findings table:** ID, severity, confidence, evidence, boundary, one-line result.
5. **Each confirmed finding:** the chat format above, plus variants, conditions, and priority rationale. Add an OWASP Top 10:2025 category only when one clearly fits; a partial audit never claims full OWASP or ASVS coverage.
6. **Needs validation:** title, trace, exact blocker, and safe owner check, with no severity.
7. **Retracted alerts, hardening notes, and good patterns found.** Include "Reviewed and cleared" for workflows, webhooks, or areas checked and found safe, with the reason.
8. **Coverage:** counts by status, important exclusions, all `deferred` and `out_of_scope` units, the final critic result, and for each scanner its version, rule set, and outcome. Never say or imply the code is secure; say what was assessed. A partial audit never claims full OWASP or ASVS coverage.
9. **Unverified candidates**, if any, by fingerprint and unit, clearly labeled as not findings.
