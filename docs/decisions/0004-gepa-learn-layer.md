# ADR-0004: GEPA is the LEARN optimizer for the dev loop

- **Status:** accepted (2026-06-23)
- **Supersedes:** —

## Context

The Otta flywheel is SENSE → SCORE → GOVERN → ACT → **LEARN**. The first four
stages run today (the `/otta-build` pipeline + Pulse measurement). LEARN — making
the loop improve its own prompts from outcomes — was the far layer, gated on
accumulating enough labeled examples for a scalar optimizer (MIPROv2 wanted ~50+).

The `/otta-build` pipeline (builder → reviewer → qa → devops) already emits **rich
textual verdicts**: "GAPS: AC2 missing at billing.tsx:40", "AC3 FAILED — no test",
"acceptance-block FAIL". That feedback is thrown away by scalar optimizers.

## Decision

**The dev loop's LEARN optimizer is [GEPA](https://arxiv.org/abs/2507.19457)**
(`dspy.GEPA`) — a reflective optimizer whose metric returns `{score, feedback: str}`
and which rewrites prompts by reflecting on full execution traces. It is
sample-efficient (paper: 35× fewer rollouts than RL, beats MIPROv2 +10%), so LEARN
can start from ~20–50 fed-back examples instead of waiting for 50+ scalar labels.

- **Optimizer by feedback shape:** dev loop → **GEPA** (rich feedback); scalar
  non-dev loops (SEO rank-delta) → **MIPROv2**, or GEPA with synthesized feedback.
- **GEPA tunes prompts** = the LLM-judgment steps (reviewer/qa/scorer). It does
  **not** touch deterministic gate scripts. Gate *thresholds* are tuned separately
  by backtest over the win/leak ledger.
- **Data capture is free** — the verdicts are a byproduct of running the pipeline;
  storing them is a ledger write, not an LM call.
- **Cadence:** weekly batch — read ledger → GEPA optimize (`max_metric_calls`
  bounded) → score on held-out valset → **keep-if-better** → promote. Per-project,
  per-loop artifacts.

## Consequences

- LEARN moves from "months away" to "prototype-able now" for dev, because the
  pipeline is already a GEPA-shaped feedback generator.
- Optimization is a controllable *batch* cost (bounded by `max_metric_calls`), not
  a per-run increase — the output is a better prompt at the same inference cost.
- Under the OAuth CLI LM, GEPA is $0 but consumes plan usage + wall-clock; optimize
  cheap single-judgment modules per rollout, never a full pipeline per rollout.
- The LEARN layer is per-project (each repo's ledger → its own optimized prompts),
  which fits the customer model (Luke's repo learns from Luke's outcomes).
