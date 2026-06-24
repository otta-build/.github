# Architecture Decision Records (ADRs)

Each ADR captures ONE decision: its context, the choice, and the consequences.

**The rule that makes this a living spec:**

- An ADR is **immutable once accepted**. You never edit the decision.
- When thinking changes, write a **new ADR** that **supersedes** the old one.
  Mark the old one `superseded by NNNN`; mark the new one `supersedes MMMM`.
- The chain of ADRs is the audit trail of *how* the architecture evolved.

The **living spec** (`docs/vision/*.md`) reflects the CURRENT state and is edited
freely via PR. Its Decision Log points to the ADRs that justify the current state.

| Layer | Mutability | Where |
|-------|-----------|-------|
| living spec | edit freely (PR) | `docs/vision/` |
| ADR | immutable; supersede | `docs/decisions/` |
| decision log | append-only index | end of each living spec |

Status values: `proposed` · `accepted` · `superseded by NNNN` · `deprecated`.
