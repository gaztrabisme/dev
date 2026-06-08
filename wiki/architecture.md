# Architecture — Personal Dev Harness

> Summary layer. Full detail: `research/00-design-v2.md`. Pillars grounded in `research/01..08`.

## What it is

A personal AI-engineering dev harness, replacing the pure-instruction `dev` skill. Goal: process enforced
by **code** (not advisory prose), context kept coherent via a **git-shaped branch/promote** model, memory
that compounds at ~zero marginal cost on **local models**, and a harness that **evolves itself** under a
constitution. Forever-personal, local-first, hardware-specific, eventually self-building.

## Three planes

- **Work** — a durable BOARD of tickets; each has a WORKPAD (Plan / Acceptance / Validation / Notes /
  Confusions) and moves through an explicit STATE MACHINE whose transitions are un-skippable gates;
  promotion to mainline = a LAND gate gated on proof-of-work.
- **Execution** — a COORDINATOR dispatching WORKERS in **git-worktree-isolated** workspaces; churn lives/dies
  in the worktree, only the squashed result + workpad promote. Model-per-role.
- **Memory** — curated **wiki** prose (semantic+procedural, never decays) + a decaying embedded **sidecar**
  (episodic), embeddings on local **oMLX** (~zero cost). Top-k retrieval, never whole-store injection.

## The work spine

```
Todo → Align → In Progress → Verify → Review → Land → Done    (+ Rework: hard reset → re-enter Align)
```
- **Align (P1)** — the keystone gate: block all execution tools until the plan + criteria are confirmed.
  Validated at runtime in the spike (`research/07`).
- **Verify** — mechanical checklist + the **Harden gate**: test quality is a *number* (mutation score),
  not a vibe. Author and mutator on different providers.
- **Land** — squash-merge; main only ever sees distilled output.

## Five pillars (+ P0)

P0 auto-context priming · P1 Align gate · P2 branch/promote isolation · P3 memory · P4 parallel exploration
with pruning · P5 reflection/self-evolution under a **constitution (immutable) / case-law (agent-editable)** split.

## Cross-cutting principles

1. **Gate by a number or an artifact, never a vibe.**
2. **The Align criteria are the keystone** — they are the Verify oracle, the P4 pruning function, *and* the
   grounding for reflection.
3. **One pipeline reused three times:** raw→distilled→promoted-with-verification = memory (episodic→wiki) =
   context (churn→main) = evolution (case-law→constitution).

## Runtime / base — UNDER EVAL

The spike validated **EXTEND pi-coding-agent via its TS ExtensionAPI** (gate = `tool_call` block). New
direction: **Rust-native** harness (Gary's call — local-first, owned forever, KB MCP already Rust). This
is being pressure-tested now (`research/10-eval-scope.md`): language strategy A (full Rust) / B (Rust core +
TS orchestration, the omp model) / C (stay TS-on-Pi). Resolution lands in `research/11-foundation-decision.md`.
Until then, §2/§10 of design-v2 (the TS adoption layer) are provisional.
