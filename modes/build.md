# Build Mode

Feature builds via coordinated subagents with test-driven development. YOU are the coordinator.

---

## Phase 1: Scope

| Signal | Weight | Process |
|--------|--------|---------|
| "Fix [bug]" / "Update [small thing]" | Light | Define done → Build+Test → Verify |
| "Add [feature] to [existing]" | Medium | Define done + interface shapes → Build+Test → Verify → Adversarial Review |
| "Build an app/service/API" | Heavy | Define done + contracts → Phased Build+Test → Verify per phase → Adversarial Review |
| Unfamiliar domain or tech | +Research | Spawn research subagent when unknowns surface |
| ML/CV/NLP task | +ML Heuristics | Read `references/ml-heuristics.md` — reframe problem before choosing approach |
| Retrieval / RAG / semantic search task | +RAG Heuristics | Read `references/rag-heuristics.md` — chunking, hybrid search, rerank, enrichment, eval |

**Knowledge Base Grounding Gate:** if the build touches a KB-covered domain (ML, DB, security, distributed systems, crypto, RAG — the `+ML`/`+RAG` rows and any security/DB-internals work), apply the gate in `SKILL.md` before implementing: search the KB and record `KB: searched "<q>" → <finding|nothing relevant>` in Key Decisions. Skippable only by stating the domain isn't covered.

---

## Phase 1.5: TDD Decision

Not every change benefits from test-first development. Apply TDD when the value of catching regressions exceeds the cost of writing tests.

| Signal | TDD? | Why |
|--------|------|-----|
| New behavior (endpoint, service, component) | YES | Tests define the contract |
| Modifying existing behavior | YES | Tests catch regressions |
| Deleting code / removing features | NO | Verify existing tests still pass |
| Mechanical changes (add classes, rename across files) | NO | Build verification is sufficient |
| Config changes | NO | Smoke test is sufficient |
| Pure frontend UI (no logic) | NO | Visual verification, build check |

When skipping TDD, state why. "Skipping TDD: deletion-only change, existing tests cover regression" is valid. Silence is not.

---

## Phase 2: Align

**Light (all projects):** Success criteria only — verifiable, binary, specific.

```markdown
## Done When
- [ ] GET /users returns 200 with JSON array
- [ ] All tests pass with 0 failures
```

**Greenfield builds** without existing code → create data schemas and API contracts as refinable coordination tools. Claude generates these from architecture docs. Track contract drift with a changelog:

```markdown
## Contract Changelog
<!-- Append entries here if contracts change during build -->
<!-- Format: - [PHASE-N] Changed: X → Y. Reason: Z. Approved by: user/coordinator. -->
```

**Extending existing code** → the codebase IS the contract. Read it, don't re-specify it. No changelog needed — the diff is the audit trail.

**Test naming convention:** `test_<contract_clause>_<behavior>` (e.g., `test_api_get_users_returns_json_array`). When a test fails, the name tells you which requirement broke.

**Before the approval gate:** If the task involves any non-obvious decisions (auth mechanism, DB schema shape, caching strategy, concurrency model, error-handling contract, etc.), surface them per `../core/references/pushback-and-teach.md`. List the forks with 2–3 options each, recommend one with a one-line tradeoff, and get an explicit answer. Do NOT proceed with silent defaults.

**STOP for user approval before building.**

### Criteria Concreteness Gate (pre-TDD)

Before spawning the test subagent, run each success criterion through this litmus:

| Question | Pass | Fail |
|----------|------|------|
| Can a stranger write a test from this criterion alone? | "Expired JWT returns 401 with `{error: 'token_expired'}`" | "Auth works" |
| Does it name a specific input → output pair? | "Empty cart returns 200 with `{items: []}`" | "Handles edge cases" |
| Would two engineers write the same test from it? | "Search with no results returns 200 + empty array" | "Search is fast" |

**Any criterion that fails → pushback loop.** Apply `../core/references/pushback-and-teach.md`: name the forks, present options, get a concrete answer. Two vague answers → stop and demand binary criteria before proceeding.

**Escape hatch:** User explicitly says "shallow tests are fine here" → document which criteria are vague and proceed. The test subagent will produce what it can; the gap is acknowledged, not hidden.

**When this gate is instant:** If criteria already read like test assertions (most concrete tasks), this adds zero overhead — you confirm they pass and move on.

---

## Phase 3: Build

### Context Management

- **Verbose commands** (builds, installs) → `bash {baseDir}/scripts/run-command.sh <command>`
- **Test runs** → `bash {baseDir}/scripts/run-tests.sh [test-path]`
- **Quick one-liners** (`mkdir`, `cp`, `mv`) → run directly

### Codebase Understanding (Existing Projects)

Before changing existing code, map its blast radius: `grep`/`rg` for every caller, importer, and test mock of the symbols you'll touch (see Signature-Change Protocol below). Read the call sites, don't guess them. For unfamiliar libraries, read the official docs / web search for the real signatures rather than inventing them.

### Signature-Change Protocol

When you edit a function's **definition** (not just its body) — parameters, return type, rename, or deletion — *before committing*, grep for all callers including test mocks:

```bash
# Example: you changed create_api_key's return type
rg -l 'create_api_key' --type py     # all files touching it
rg 'mock.*create_api_key|patch.*create_api_key' --type py   # test mocks
```

Test mocks that specify return shape (`.return_value = {...}`) are the most fragile — they duck-type against a contract that exists only in runtime, so static analysis can't warn you. Update them in the same commit as the signature change. If you can't (too many callers, out of scope), create a task for each stale caller *before* marking this one complete.

Typed mocks (`MagicMock(spec=ActualClass)`, `Protocol`-based fixtures) fail loud when the real contract changes — prefer them in new tests.

### When to Use Subagents vs Coordinator-Direct

| Task Characteristics | Approach |
|---------------------|----------|
| Single file, <50 lines changed | Coordinator-direct |
| Clear scope, no ambiguity | Coordinator-direct |
| Multiple files, complex logic | Implementation subagent |
| Needs isolation from main context | Subagent (protects context window) |
| Parallel with other work | Subagent (background) |
| Mechanical across many files | Structured edit pattern (see `references/subagent-briefs.md`) |

**Declare the pattern, don't default into one** (SKILL.md → Pattern Gate). `coordinator-direct` is a legitimate *declared* choice for mechanical/unambiguous work — but when a task has independent parts or logic that can be wrong, rule out fan-out / a subagent chain *on purpose*, not by omission. Subagents cost context and coordination overhead; they earn it through parallelism, isolation, or independent-reviewer separation.

### Subagent Orchestration

**Feature builds → subagents** (test separation pays for itself). **Wiring, data prep, config, small fixes → coordinator works directly.**

Spawn in this order:
1. **Test subagent** — BEFORE implementation. Tests verify contract, not code.
2. **Test validation subagent** (opt-in) — Reviews tests cold against business requirements. Use for high-stakes domains: finance, security, healthcare, data integrity. Skip when the test subagent wrote tests against a clear spec — both agents read the same requirements and have correlated blind spots.
3. **Implementation subagent** — Write code that passes tests.
4. **Verification subagent** — Tests pass, quality scan, contract compliance, no mocked production.
5. **Adversarial review subagent** (medium/heavy only) — Cold code review, no planning context.

Read `references/subagent-briefs.md` for exact prompt templates. When briefing subagents, provide ONLY the relevant prompt constraints from that file plus task-specific context. Do not forward skill-level instructions.

**Worker VCS hygiene (non-negotiable when subagents write code).** Workers produce file changes; the **coordinator** owns the index and the commit. No subagent runs `git add -A`/`.`, `git commit`, or any branch/reset/stash/checkout — see the isolation constraint at the top of `references/subagent-briefs.md`. When two or more implementation subagents run **in parallel**, give each its **own git worktree** so their edits can't collide or cross-stage; a shared checkout with concurrent writers corrupts silently. Reviewing and staging by explicit path is the coordinator's job, after the workers return.

---

## Phase 4: Verify

Verification subagent covers: tests pass, code quality scan (ruff + bandit + pyright via `run-quality.sh`), contracts met, no mocked production code, ai-slop-detector, success criteria evidence.

### Gate-Script Smoke Test (when the build ships its own gates)

When the deliverable mixes LLM agents with deterministic scripts that act as **gates or scaffolding** (validators, stage checks, instantiators), exercise the scripts' failure paths before trusting them: missing input, malformed input, stale input, empty input — each must fail loud with the documented output shape. The scripts get less scrutiny than the prompts but carry the integrity load; a gate that opens on garbage is worse than no gate, because it converts "unverified" into "falsely verified". (Trace: accelerator-kit 2026-07-03 — the stage gate passed with a missing service due to an untested parse path, while the agent markdown survived cold review untouched.)

**Red flags (auto-fail):** Mocked production code, tests testing mocks, modified success criteria, "it works" without proof, lint errors, HIGH/CRITICAL security findings.

### Test-Health Gate (prevents bit-rot accumulation)

Run the **full** test suite (not just the tests you added/modified) before marking complete. If the suite has ANY failures — whether caused by this task or pre-existing — triage each one:

| Disposition | When | Requirement |
|-------------|------|-------------|
| **Fix** | The test is correct; the code (or the test's mock of current code) is stale | Fix in this commit |
| **Quarantine** | The test needs non-trivial work and blocks this task | `@pytest.mark.skip(reason="...")` / `xfail` with a one-line reason that names the drift, *and* a task entry created to fix it |
| **Delete** | The test is for behavior that no longer exists or was replaced | Remove the test; note in commit message |

**"Pre-existing, not mine"** is not a valid defer. A pre-existing red test is test bit-rot — the function evolved and the test didn't. Leaving it red is how you end up with 5 broken tests that nobody noticed for weeks. Triage it or quarantine it with a ticket; silence is the failure mode.

Exception: if the user explicitly scopes the task to exclude test hygiene ("just fix X, I'll clean tests later"), document the pre-existing failures in the session summary and move on. The documentation itself is the defer.

---

## Phase 5: Adversarial Review

A separate subagent reviews the code **cold** — no planning context, no contracts, no rationale. Reads the code like a new developer joining the project.

**Run only when the trigger fires.** Cold review earns its cost when a mistake would be:
- **hard to reverse** — schema/API/data-model commitments, public contracts, anything downstream builds on; or
- **silent** — ships green and surfaces later as corruption/drift, not as a failed test; or
- **green for the wrong reason** — a success metric or passing test that could be satisfied *without the system actually working*: data leakage, memorization, a label/grid/measurement artifact, a coincidental pass.

The third case is why adversarial review is a gate **distinct from verification**, not a redundant one: a verifier checks the code against the spec and so *inherits the spec's blind spots* — it structurally cannot catch a result that's right for the wrong reason. Only a reviewer reading the artifact cold, with no spec to anchor on, catches "passes for the wrong reason." Skip only when none of the three holds (cheap-to-undo and loud-when-wrong) — and say so out loud (a silent skip is an integrity miss).

**Keep it cold.** The reviewer gets the artifact **context-free** (file paths only — no spec, no rationale, no "why we chose X") and must **not** be the author of the code or of the tests being reviewed — author and reviewer share correlated blind spots. Test-author ≠ implementer ≠ reviewer.

- **CRITICAL findings** → must fix before delivery
- **3+ CRITICAL** → systemic problem, STOP and escalate to user

---

## Gate Enforcement

When a plan specifies gates (adversarial review, test validation, verification), they are NOT optional during execution.

Before marking any item complete:
1. Check what gates were planned
2. Execute each planned gate
3. If skipping a gate: state the reason explicitly ("Skipping adversarial: light-weight config change, 3 lines modified")

The coordinator must not skip gates silently. Planned gates are promises to the user.

---

## Evidence

Match effort to task weight:

| Weight | Evidence |
|--------|----------|
| **Light** (bug fix, config, wiring) | Pass/fail count + 1-line summary per criterion |
| **Medium** (feature, refactor) | `run-tests.sh` JSON in `logs/` + checked criteria |
| **Heavy** (multi-phase, greenfield) | Full `logs/` directory + report.md + checked criteria |

Minimum: every success criterion either checked with evidence, deferred with reason, or marked N/A.

**Output contract:** before reporting the build complete, append the outcome (with evidence pointer) to `wiki/log.md`, record any non-obvious choice in `decisions.md`, and file new failure patterns in `gotchas.md`. The build isn't done until that write exists — see `../core/references/wiki-protocol.md` → Output Contract.

---

## Failure Protocol

```
Attempt 1: Same subagent retries with error feedback
Attempt 2: Fresh subagent with original spec + "Approach X failed, try Y"
Attempt 3: ESCALATE to user
```

Escalation includes: what was attempted, specific blockers, options (simplify scope / change approach / user assists / abort).
