# ADR-0003: Lifecycle linkage rides the PR body; Pulse derives it server-side

- **Status:** accepted (2026-06-23)
- **Supersedes:** —

## Context

The idea→issue→PR→version chain needs an origin (`idea_ref`) and cross-system links
(`Fixes #N`) that GitHub webhooks carry but don't surface. The first cut used an
authenticated `POST /event` with a shared webhook secret — which external customers
(e.g. Luke) never receive, so ship-tracking only worked first-party.

## Decision

**The linkage lives in the PR body** (`idea_ref:` + `Fixes #N`), enforced by the CC
plugin's shipping loop. **Pulse derives `issue_shipped` server-side** from the
`pull_request` merged webhook it already receives — no `/event` POST, no shared secret
on the customer's machine. Version resolves at read time by joining the PR to the
`deploy_tag` annotation.

## Consequences

- Ship-tracking works for every install, including external users — the unlock for
  selling Pulse (see [[0002-pulse-is-the-product-spine]]).
- `POST /event` remains for first-party/admin use; per-install token auth is a later
  option (issue #25), not required for the default path.
- The plugin gate requires `idea_ref` + `Fixes #N` in the body, so the contract Pulse
  reads is guaranteed.
