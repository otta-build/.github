# ADR-0006: Verification quality is a human+LEARN layer, not a green checkmark

- **Status:** accepted (2026-06-24)
- **Supersedes:** —
- **Relates to:** ADR-0004 (GEPA LEARN), ADR-0005 (App gate)

## Context

The `/otta:dev` run for issue #58 (the session-insight miner) passed every stage —
8 unit tests, COMPLIANT spec review, 5/5 ACs verified, gate green. Then running the
shipped code on a **real** session showed the core heuristic (human-intervention
tagging) mis-classifying skill definitions, hook output, system reminders, and bash
I/O as "human interventions" — ~10 of the first 12 were noise.

The lesson: **the tests graded the code against ACs we wrote, on a fixture we
designed.** Green proves "code matches the spec," never "the spec was right" or
"the heuristic works on real data." It is a self-graded exam. This generalizes to
every project/customer — you cannot author perfect ACs/heuristics upfront for a
codebase you don't control.

A second, related gap: the loop did not know the project's **delivery context**, so
"deployed?" was fuzzy and CI presence was assumed. The loop guessed because nothing
declared it.

## Decision

Verification quality is handled by **four layers**, and human judgment is a
**permanent** layer, not a temporary crutch:

1. **Verify on real data in-loop.** For any heuristic/judgment-heavy AC, the
   `otta:qa` stage runs the code on a **real sample from the project** and surfaces
   the output — not only the author's fixture. (The 30-second check that caught #58.)
2. **Human at two points: spec-write and merge-approve.** The ACs *are* the spec;
   the loop cannot validate its own spec (circular). The machine automates
   execution + verification-*against*-spec; the human owns is-the-spec-right and
   does-it-work-on-real-data. Green ≠ approval.
3. **Escaped defects are ground truth.** A green verdict later contradicted by
   `issue_reopened`/`pr_reverted` is a labeled false-positive — the only objective
   signal that verification was wrong (captured per ADR-0005 events).
4. **LEARN tunes gates per project.** Gates start generic (from `.otta.yml` +
   defaults) and graduate from *that project's* ledger + escapes (ADR-0004 GEPA).
   Verification learns the codebase instead of being hardcoded for it.

### Delivery context — `.otta.yml`

Every repo declares its delivery model so the loop stops guessing:

```yaml
base: main
staging: staging            # or null (straight-to-prod)
deploy:
  mode: auto-on-merge       # | tag | manual | none
  target: <what "deployed" means here>
  package_paths: [apps/pulse/**]
ci:
  required: true
pulse:
  installed: true
```

`/otta:setup` detects what it can (branches, CI, deploy webhooks), asks the few it
can't, onboards the App, and commits `.otta.yml`. From then on the loop's
"deployed?" answer is real and the gate (ADR-0005) reads it.

## Consequences

- `otta:qa` gains a **real-sample dry-run** step for heuristic ACs.
- `/otta:dev` must **pause at genuine decisions** (spec forks, heuristic design) —
  front-loading them defeats developer-in-the-loop (the #58 run did exactly this).
- The same generic loop serves different customers via per-repo `.otta.yml` +
  per-repo LEARNed gates — no per-customer hardcoding.
- Human approval frequency shrinks as LEARN improves, but never reaches zero.
