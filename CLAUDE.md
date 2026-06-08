# Project Context — Personal Dev Harness

> Project-level context for this repo (distinct from Gary's global `~/.claude/CLAUDE.md`). Auto-loaded each
> session — kept thin on purpose. **Detail and current state live in `wiki/`; this file is the map + the rules.**

This repo is being re-platformed from a **pure-instruction `dev` agent skill** into a **personal AI-engineering
dev harness**: process enforced by code, context kept coherent via git-shaped branch/promote, memory that
compounds at ~zero cost on local models (oMLX), and a harness that eventually builds and evolves itself.
**Direction: Rust-native, local-first, hardware-specific, forever-personal** (language strategy under eval —
see `wiki/decisions.md`).

## Start here (session protocol — do this first)

1. Read `wiki/index.md` → `wiki/active-work.md`. Brief the user on current status + last breadcrumbs.
2. Check `wiki/decisions.md` before re-opening a settled question (incl. its **Rejected Approaches**).
3. Then work.

## Workspace map

**This repo** (`Skills/dev/`):
- `SKILL.md`, `modes/`, `references/`, `scripts/` — the **original `dev` skill** (the judgment layer:
  ml/production/rag heuristics, pushback-and-teach, wiki-protocol). Good content, being ported to the harness.
- `EVOLUTION.md` — the skill's self-evolution log.
- `research/00..10` — research foundation + design + eval scope (point-in-time artifacts; see `wiki/index.md`).
- `wiki/` — **living project memory. Read first, keep updated.**

**Siblings** (outside this repo, not git-tracked here):
- `../harness-spike/` — the validated Pi spike: `align-gate.ts` (the `tool_call` gate, proven at runtime),
  oMLX provider config, `run-pi.sh`/`drive-tmux.sh`.
- `../pi-mono/` — Pi clone (`badlogic/pi-mono`), runtime reference.
- `../symphony/` — Symphony clone (`openai/symphony`), coordinator reference.

## Conventions (the dev instruction convention)

- **Keep the wiki updated** (per `references/wiki-protocol.md`): during work, add breadcrumbs/decisions to
  `active-work.md`; on completion, append to `log.md` and update `index.md` (new pages) + `decisions.md`
  (decisions *and* rejected approaches). Breadcrumbs over summaries (specific findings, not "looked at X").
- **Align before execute.** For non-trivial work, gather context, state assumptions + success/failure
  criteria, and confirm before acting. We are *building* this gate — dogfood it.
- **Gate by a number or an artifact, never a vibe** (tests, proof, checklists — not opinions).
- **Integrity constraints (from `SKILL.md`, override everything):** never modify success criteria to fit
  the result; never report success without evidence; never fake results; if stuck >3 attempts, STOP and report.
- **Wu Wei:** is this actually causing a problem (blocking work, bugs, maintenance)? If not, drop it.
- **Dependencies are liabilities.** Standard-library / own-it over wrappers; the bar for a dep is high
  (reinforced by the Rust-native direction — we mine external refs for *designs*, not runtime deps).
- **Pushback-and-teach:** challenge vague/hand-wavy asks, surface the real decisions, narrate the *why*
  (the user is learning the stack through this work).
- **Commit discipline:** only when asked; stage explicit paths (never `git add -A`/`.`); don't stage
  `.DS_Store`; end messages with `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- **Local model:** oMLX at `http://127.0.0.1:8000/v1` (key `73067799`) for local/cheap roles; model-per-role.

## Current focus

→ `wiki/active-work.md` — the **Foundation Eval** (DR1 oh-my-pi as Rust reference, DR2 beads schema) that
locks the language/runtime strategy and gates Phase 0.
