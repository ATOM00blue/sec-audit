# Variant hunting

Adapted from Trail of Bits' `variant-analysis` skill (CC BY-SA 4.0). See `../NOTICE.md`.

Start from one known bug and find the others. One root cause usually has several forms, and they often aren't in the module where the first turned up. Don't use this without a bug in hand.

## 1. State the root cause

Answer four questions: what operation is dangerous, what data makes it dangerous, what's missing, and what context enables it. Then write one sentence:

> This exists because [UNTRUSTED DATA] reaches [DANGEROUS OPERATION] without [REQUIRED PROTECTION].

For a logic bug with no data flow, state the broken invariant: "this must return false for unauthenticated callers, and it returns true when both IDs are null."

Knowing why the bug exists tells you where to look. A copy-paste bug clusters in sibling files. An API misuse appears at every call site of that API. An incomplete fix leaves its siblings unfixed.

## 2. List the search axes

Each axis goes to its own search, so it must name real constructs, not themes. Grep to confirm each exists here before keeping it.

- **Related identifiers:** names playing the same role (`ownerId`, `creatorId`, `customerId`).
- **Other shapes of the mistake:** inverted conditions, wrong fall-through return, `or` for `and`.
- **Data-type edges:** both sides null, empty string versus null, zero versus null, empty collections.
- **Name and behavior mismatch:** functions named `check`, `validate`, `deny`, or `require` whose return value means something else.
- **Parallel paths:** the same effect through batch, retry, legacy, admin, cron, webhook, or stored procedure routes.

Axes must not overlap. Two that find the same code waste an agent.

## 3. Prove the pattern on the known bug

Write a pattern matching only the known instance and confirm it hits that exact line. If it finds nothing, or hits only other lines, stop and fix the root cause. Everything after this is calibrated against it. Single-quote every pattern in shell commands, because patterns built from code can contain `$(...)` and backticks.

## 4. Generalize one step at a time

Change one element, run it from the **repo root**, read every new match, then decide whether to continue. Changing several elements at once leaves no way to tell which one caused the noise.

| Level | What it matches | Typical use |
|---|---|---|
| 0 Exact | the literal code | confirming a fix |
| 1 Variable | same code, any names | copy-paste bugs |
| 2 Structural | same shape, different surroundings | auditing a component |
| 3 Semantic | source reaches sink without protection | full assessment |

Use `rg` first. Use `semgrep` or `codeql` for structure and data flow only if already installed. Stop climbing when more than about half the matches are noise; go back a level and try a different abstraction. A sweep that adds nothing new ends that axis.

Searching only the module where the bug was found is the most common reason a hunt comes up empty.

**Scale.** Count the primary language's source files, leaving out vendored and fixture trees. At about 40 files or fewer, one agent covers two axes in a single pass, because fan-out buys nothing. On larger trees, run up to 6 axes per round, one agent each, with up to 2 rounds, or 4 if the user asks for depth. Rotate through unswept axes on later rounds. Stop after 2 rounds in a row add nothing confirmed.

**Report how it stopped:** swept until dry, sized to a small tree, or hit the round or budget limit while still finding variants. Name any axis never swept under its own heading. A bounded sweep must not read as an exhausted one.

## 5. Triage each match

Read the function, its callers, and the types involved. Look for what makes it safe: an earlier guard, middleware, a sanitizer, a parameterized API, a type constraint, a caller set that never passes attacker input. Then establish three things:

- **Reachable** from an external entry point?
- **Controllable** by an attacker?
- **Unprotected:** is the protection named in the root cause really missing here?

Reachable and controllable with a different protection in place is a false positive worth recording. Unreachable today but unprotected is a real finding at lower severity; say what would make it reachable.

Give every surviving site to a verifier (see `agent-prompts.md`), in batches of up to 4. Establish each site's conditions and impact separately, because a sibling can be far worse than the original.

Every candidate gets an explicit answer: a real instance, or ruled out with the guard named. "Worth a look" isn't an instance, and "can't rule it out" isn't a ruling. Check a guard is real before accepting it: `startsWith` on a string path can still be escaped. A different weakness at a site that's safe from this bug is a separate finding, not a variant.

## 6. Write it up

Include all of these in the report:

- The root cause sentence, the original location, and the code as quoted from the file.
- **Search table**, including patterns that failed:

  | Pattern | Tool | Level | Matches | Real | False |
  |---|---|---|---|---|---|

- Confirmed sites, worst first, with quoted code.
- False positives grouped by reason, not one row per match. If there are no real variants, say so explicitly and show what was searched. Negative coverage is a result.
- Immediate fixes, plus a regression guard: a CI check or lint rule from the pattern that found the most, or a named test if no pattern is precise enough.
