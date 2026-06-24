# ADR-0002: Otta Pulse (the GitHub App) is the product spine

- **Status:** accepted (2026-06-23)
- **Supersedes:** —

## Context

Otta has several surfaces (Cockpit desktop app, CC plugin, landing). GitHub shipped
the Copilot App, which commoditizes the cockpit/agent-IDE surface. We need to know
which layer is defensible and therefore the spine.

## Decision

**The GitHub App (Otta Pulse) is the product spine.** Only a GitHub App can *enforce*
governance on GitHub (Checks API is App-exclusive) and observe DORA + the idea→version
lifecycle via webhooks. Lead with "install the Otta Pulse App on your repo," not
"download the desktop app." Every other surface (plugin, Cockpit, Hermes) is a *client*
of the same Pulse backend.

## Consequences

- Cockpit is repositioned as the governance + observability + learning layer over Pulse,
  not another agent editor (the editor is commoditized).
- Distribution leads with the App (PLG: install → DORA backfill in minutes).
- The CC plugin stays thin: it enforces the shipping discipline and feeds Pulse via the
  PR body; it is not the product.
