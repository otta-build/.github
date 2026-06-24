# ADR-0001: GitHub is the canonical tracker; Linear is a mirror

- **Status:** accepted (2026-06-23)
- **Supersedes:** —

## Context

Work was being tracked in both Linear (OPS team) and GitHub issues, drifting apart.
A single source of truth is needed for issues, roadmap, and releases.

## Decision

**GitHub is canonical** for issues, Projects (roadmap), milestones (versions), PRs,
checks, releases, and deployments. **Linear/Jira are replaceable adapters** — a
read-only mirror/triage view at most, never the source of truth. When GitHub and a
tracker disagree, GitHub wins.

## Consequences

- Roadmap lives as a GitHub Project + milestones. Issues `#21–#30` are canonical.
- Linear OPS-* issues mirror or retire for Otta scope; they don't drive work.
- Tooling (Otta Pulse, the plugin) reads GitHub natively — no tracker adapter needed
  for the core loop.
