# dev — Personal Dev Skill (instruction)

> Project-level context for this repo (distinct from Gary's global `~/.claude/CLAUDE.md`). Kept thin.
> This repo is **instruction content, not a runtime** — a pure-instruction agent skill, maintained and
> evolved over time.

The `dev` skill is the **judgment layer** for development work: design / build / sprint / assess / train /
evolve playbooks, plus ml/production/rag heuristics. The shared spine (pushback-and-teach, the wiki
protocol, the grounding gate, the evolution loop) is inherited from the `core` kernel (`../core/references/`). Invoked as
`/dev` or proactively by mode detection (see `SKILL.md`).

These same references are the **source material being ported into the separate harness project**
(`../harness/` — the Rust-native dev harness that makes this process load-bearing instead of advisory).
The harness mines this skill for content; it does not fork it. The two evolve on independent tracks.

## Structure
- `SKILL.md` — entry point: principles, integrity constraints, mode detection, the Grounding Gate.
- `modes/` — design / build / sprint / assess / train / evolve playbooks.
- `references/` — ml-heuristics, production-thinking, rag-heuristics, subagent-briefs. (Shared spine — pushback-and-teach, wiki-protocol — now lives in `../core/references/`.)
- `scripts/` — run-tests / run-quality / run-command / analyze.py (JSON-summary wrappers).
- `EVOLUTION.md` — the skill's self-evolution log.

## Conventions for editing this skill
- **Keep it instruction, not code.** Elegant / clean / lean. Wu Wei — add structure only when its absence
  causes a real failure, not for theoretical completeness.
- **Evolve via the `evolution` skill** (`../evolution/SKILL.md`): generalized lessons (reusable beyond one
  project), git-committed, human-gated, size-budgeted. Evolve stopped being a dev mode on 2026-08-09 —
  the machinery served every skill, not just this one.
- **Commit discipline:** only when asked; stage explicit paths (never `git add -A`/`.`); don't stage
  `.DS_Store`; end commit messages with `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`.
- The harness project (Rust code, its wiki, its research) lives in `../harness/` — **not here.**
