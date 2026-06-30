# Wiki Protocol

Persistent project wiki that compounds understanding across sessions. Replaces session-scoped work tracking with a knowledge base Claude maintains directly as markdown.

Inspired by Karpathy's LLM Wiki pattern: knowledge is compiled once and kept current, not re-derived every session. The human curates and directs; Claude does the bookkeeping.

---

## Three Layers

1. **Raw sources** — The codebase, docs, configs, data files. Immutable — Claude reads but never modifies these as part of wiki operations.
2. **The wiki** — `wiki/` directory of Claude-maintained markdown. Summaries, entity pages, decision records, work status. Claude owns this entirely.
3. **The schema** — This protocol file. Tells Claude how to maintain the wiki.

---

## Location

The wiki lives at `wiki/` in the project root (visible in Obsidian, git-trackable). Symlink from `.claude/wiki` for Claude Code native access:

```bash
# First session initialization
mkdir -p wiki
ln -sf "$(pwd)/wiki" .claude/wiki
```

Ask the user: add `wiki/` to `.gitignore` (private) or track it in git (team-shared)?

---

## Wiki Structure

```
wiki/
├── index.md          # Catalog of all pages — one-line summary each
├── log.md            # Append-only chronological record
├── architecture.md   # System architecture understanding
├── decisions.md      # Key decisions and rationale
├── active-work.md    # Current workstreams, status, next steps
└── [topic].md        # Entity pages, concept pages, investigation notes — as needed
```

### index.md

Content-oriented catalog. Every wiki page listed with link + one-line summary, organized by category. Update on every ingest or page creation.

```markdown
# Wiki Index

## Architecture
- [architecture.md](architecture.md) — System overview, component map, data flow

## Decisions
- [decisions.md](decisions.md) — Why we chose X over Y, constraints, tradeoffs

## Active Work
- [active-work.md](active-work.md) — Current workstreams, blockers, next steps

## Topics
- [auth-system.md](auth-system.md) — Authentication flow, token lifecycle, session management
```

### log.md

Append-only. Each entry prefixed consistently for grep-ability:

```markdown
# Wiki Log

## [2026-04-08] session | Initial setup
- Created wiki, ingested codebase architecture
- Started auth-system refactor workstream

## [2026-04-09] session | Auth refactor progress
- Completed token rotation implementation
- Discovered session store race condition — added to active-work.md
```

### active-work.md

Replaces `.tracks/` node files. Each workstream is a section:

```markdown
# Active Work

## Auth System Refactor
**Status:** In progress
**Started:** 2026-04-08
**Goal:** Replace session-token storage for compliance

### Current State
- Token rotation implemented, tests passing
- Session store race condition discovered — needs investigation

### Next Steps
- [ ] Investigate race condition in concurrent session invalidation
- [ ] Add integration tests for token refresh flow

### Breadcrumbs
- 2026-04-09: `SessionStore.invalidate()` not atomic — wraps Redis MULTI but doesn't handle partial failure
- 2026-04-08: Existing tests mock the session store — need real Redis tests
```

### decisions.md

Each decision is **Context → Decision** (why the alternatives lost, the constraint, the tradeoff). Two categories are easy to skip but carry the most future value:

- **Rejected Approaches** — things you evaluated and deliberately did *not* adopt, with *why* (cost, marginal gain, complexity). This is what stops a future session re-litigating a settled question or re-trying a dead end. Keep a standing "Rejected Approaches" section.
- **Post-mortems on reverted decisions** — when something shipped and was then pulled, or a whole direction lost its gate, record a **"why it failed — the lesson"** note explaining the *mechanism*, not just the outcome. A negative result documented this way is reusable knowledge; an undocumented one gets repeated.

```markdown
## Rejected Approaches
### GraphRAG / LightRAG
**Why considered:** cross-document concept linking.
**Why rejected:** needs another full LLM pass; marginal gain for single-hop lookups.

## <Direction> — REVERTED (date)
**Finding:** [the decisive number that killed it]
**Why it failed — the lesson:** [the mechanism, so nobody re-runs it]
```

### Other pages that earn their place

`log.md` and `active-work.md` are defaults, not mandates. Real projects also sustain pages like `gotchas.md` (failure patterns + watch-fors) and `ops-runbook.md` (the exact commands to run recurring operations) — create them when the content recurs. Pages earn existence by being referenced; don't force the prescribed set if the project's shape calls for different ones.

---

## Operations

### 1. Ingest (First Encounter or New Major Area)

When entering a project for the first time or exploring a new area:

1. **Check for existing wiki** — Read `wiki/index.md`. If it exists, skip to Session Protocol.
2. **Initialize** — Create `wiki/` directory, symlink, create index.md and log.md.
3. **Build understanding** — Read codebase structure, key files, existing docs. Create:
   - `architecture.md` — Component map, data flow, key abstractions
   - Entity pages for major components (as needed)
   - `active-work.md` — Current state of any in-progress work
4. **Update index.md** — Add all new pages.
5. **Log** — Append ingest entry to log.md.

**Don't over-ingest.** Create pages for what you've actually read and understood, not for everything that exists. Pages grow organically as you work.

### 2. Session Protocol (Every Session)

On entry to any dev mode:

1. **Read** `wiki/index.md` → `wiki/active-work.md` (+ `decisions.md` when you're about to make a call). **Do not read `log.md` or topic pages wholesale — grep them on demand.** The read-on-entry set is what costs context *every* session; keep it to the current-state pages (see Compaction).
2. **Brief the user** — Current status, last breadcrumbs, pending items
3. **During work:**
   - Update `active-work.md` with findings, decisions, breadcrumbs
   - Create/update topic pages when you learn something worth preserving
   - Cross-reference: when page A mentions a concept that has page B, link them
4. **On completion:**
   - Update relevant wiki pages with outcomes
   - Append session entry to `log.md`
   - Update `index.md` if new pages were created

### 3. Lint (Periodic or On Request)

Health-check the wiki. Run when user asks, or suggest after 10+ sessions.

Check for:
- **Stale pages** — Last updated many sessions ago. Still accurate?
- **Contradictions** — Page A says X, page B says Y
- **Orphan pages** — In index but never linked from other pages
- **Missing pages** — Concepts referenced but lacking their own page
- **Active-work decay** — Workstreams with no updates. Still active or done?
- **Oversized pages** — Any read-on-entry page (`active-work.md`, `architecture.md`) past ~400 lines → compact it (see below).

Output: list of findings, suggested actions. User approves which to fix.

---

## Compaction — the cost is what you read, not what you store

A wiki that grows to thousands of lines (a sprawling `log.md`, a bloated `active-work.md`) is only a problem if you *read it wholesale every session*. The fix is discipline about the **read-on-entry set**, not a database or an external memory tool (those typically inject retrieved memory into the prompt each turn, which busts prefix caching and burns tokens — a plain markdown wiki read once at session start stays cached).

- **Read-on-entry budget.** On entry, read only `index.md` + `active-work.md` (+ `decisions.md` when deciding). Everything else — `log.md`, topic pages — is **grep-on-demand**. A 2,000-line `log.md` costs nothing if you never read it in full.
- **Compact current-state pages; append only journals.** `active-work.md` and `architecture.md` are *living* documents — rewrite and prune them, don't append. When a workstream finishes, collapse its section to a one-line `log.md` entry (+ a `decisions.md` entry if a choice was made) and **delete it from `active-work.md`**. The only page that grows unbounded is `log.md`, the one you don't read wholesale.
- **Size trigger.** When a read-on-entry page passes ~400 lines, compact it: fold resolved detail into a terse summary, push specifics into a dated `log.md` archive or a topic page. Bloat in a read-on-entry page is a tax on every future session.
- **Bootstrap so re-init survives.** Put a one-liner at the top of the project's `CLAUDE.md`/`AGENTS.md`: *"On entry: invoke the dev skill; read `wiki/index.md` + `wiki/active-work.md`."* This re-initializes the memory workflow on every fresh session or context compaction — without it, a compaction silently drops the wiki habit and the next turn works blind.

---

## Output Contract (the completion gate)

**No mode is done until it has written its result to the wiki.** Completion is gated by the *artifact*, not by the model's sense that the work is finished — a mode that did the work but wrote nothing down has not completed, and should not be reported as complete. Before claiming done, name the file and confirm it exists.

Every mode, at minimum, appends a `log.md` entry (what happened, dated) and updates `active-work.md` (current state + next steps). Beyond that floor, each mode has a **primary artifact** it must produce:

| Mode | Primary artifact (must exist before "done") |
|------|---------------------------------------------|
| **Design** | A spec/design page (the contracts, success criteria, data models) **+** a `decisions.md` entry recording what was chosen and what was rejected and why. |
| **Build** | `log.md` entry with the outcome + evidence pointer; a `decisions.md` entry **if** a non-obvious choice was made; any new gotcha → `gotchas.md`. |
| **Sprint** | `log.md` batch summary + per-item status reconciled in `active-work.md` (done / deferred / blocked). |
| **Assess / Analyze** | A findings page (e.g. `assessment-<date>.md` or `analysis-<topic>.md`) — the findings, severities, and recommended actions — linked from `index.md`. |
| **Train** | An experiment-log entry (config, metric vs. pre-registered baseline, verdict) + `decisions.md` if the result changes direction. A negative result is a required write, not an optional one. |
| **Evolve** | The evolution-log entry (this skill: `EVOLUTION.md`; other projects: `decisions.md` + `log.md`) recording harvest scope, patterns, and applied changes. |

**Close-out checklist (run before reporting any mode complete):**
- [ ] `log.md` entry appended (dated, grep-prefixed)
- [ ] `active-work.md` reflects true current state (no stale "in progress" for finished work)
- [ ] Primary artifact for this mode exists and is linked from `index.md`
- [ ] Decisions *and* rejected approaches recorded where one was made
- [ ] Breadcrumbs are specific findings, not "looked at X"

The cost of this contract is a few minutes of writing; the cost of skipping it is every future session re-deriving what this one already knew. That asymmetry is why it's a gate, not a suggestion.

---

## Principles

- **Wiki is Claude's memory, not the user's docs.** It captures what Claude learned about the project — architecture understanding, decision rationale, work state. It's not user-facing documentation (that's in the project's own docs).
- **Compile once, update incrementally.** Don't re-read the entire codebase each session. Read the wiki, work, update what changed.
- **Breadcrumbs over summaries.** Record specific findings ("SessionStore.invalidate() not atomic") not vague summaries ("looked at session code").
- **Pages earn their existence.** A page should exist because it's been referenced or will be referenced. Don't create pages speculatively.
- **Cross-reference aggressively.** The value of a wiki is the links. When you mention a concept that has a page, link it.
