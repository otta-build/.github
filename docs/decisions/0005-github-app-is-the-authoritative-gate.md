# ADR-0005: The GitHub App is the authoritative gate

- **Status:** accepted (2026-06-24)
- **Supersedes:** —
- **Extends:** ADR-0002 (Pulse is the product spine)

## Context

Governance today is enforced by a **local pre-push gate** (`otta-gate.sh`) plus
whatever per-repo CI a project happens to have. This is brittle and not a moat:

- The local gate is **bypassable** (`OTTA_SKIP_GATE=1`, or just don't install the
  hook) and runs only on the developer's machine.
- Per-repo CI is **hardcoded per project** — copy a workflow into every repo, keep
  it in sync, and it still varies by customer. `apps/pulse` PRs, for example, get
  the local gate but **no GitHub CI** at all.
- Neither survives the executor changing. The whole point of Otta is to gate
  *whatever* opened the PR (Claude Code / Copilot App / Codex / human).

Only a GitHub App can enforce governance *on GitHub*: the **Checks API is
App-exclusive**, runs server-side, is un-bypassable via branch protection, and is
executor-agnostic by construction.

## Decision

**The Otta Pulse GitHub App's Checks API status is the authoritative merge gate.**
The local pre-push gate is demoted to *optional fast feedback*; per-repo CI is
*subordinated*, not the source of truth.

The App publishes ONE merge-blocking Check that aggregates:

```
Otta Gate (Check, required via branch protection) =
   CI conclusion == success        # the repo's own CI runs the tests (see boundary)
   AND acceptance-block present     # App, server-side, from the PR body
   AND lifecycle linked (idea_ref + Fixes #N)   # App
   AND coverage ≥ threshold         # App, reads the CI coverage artifact
   AND no known-escape pattern      # App, from the LEARN ledger
```

- **Centralized engine, declarative config.** The gate *logic* lives once in the
  Pulse server and runs identically for every install. Per-repo behavior is
  declared in `.otta.yml` (base/staging, thresholds, which checks are required) —
  **no hardcoded `gate.sh` per project, no copied CI workflow for governance.**
- **The moat.** The plugin is open and copyable; the App + each customer's
  LEARN-tuned gates are not. Governance enforcement and the accruing per-customer
  ledger are the defensible layer.

### Boundary — what the App CANNOT centralize

Arbitrary **test execution** cannot run on Otta's server (unsafe to execute
customer code). So tests still run in the customer's CI (GitHub Actions) or
locally. The App **subordinates** CI — it requires CI green AND adds the
governance/lifecycle/learn checks — it does **not** replace test execution.

## Consequences

- `/otta:setup` installs the App and writes `.otta.yml`; it no longer needs to
  scaffold governance CI per repo (it may scaffold a thin *test-runner* CI only if
  a repo has none).
- "No CI on `apps/pulse` PRs" stops mattering — the App Check is present on every
  repo regardless.
- Branch protection requiring the Otta Check makes the gate un-bypassable.
- Gate-engine work (Checks API + `.otta.yml`) tracked as the OPS-205 lineage.
