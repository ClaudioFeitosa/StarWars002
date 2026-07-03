# Execution Protocol — Agent Runtime Standards

## Purpose

ROUTING: This file is self-routing. Read only the sections your skill needs,
in the order given by the routing rule below — do not preload the whole file.
Each skill discovers its sections from that rule, then reads each lazily as it
becomes relevant. Sections not selected for your skill are not required.

This file defines how agent sessions manage identity, state, context limits,
recovery, section-by-section generation, and the harness output sidecar
contract. All code-development and synthesize skills follow these patterns;
the routing rule below tells you which sections apply to yours.

---

## Skill → Required Sections (routing rule)

Skills self-route. Each SKILL.md declares its own **generation shape** inline
(`section-shape` = one multi-section spec; `list-shape` = many discrete files;
`hybrid` = section-shape manifest + list-shape per-item files) and references the
exact sections it applies at each step (`apply §10.2`, `§11`, …). Route by shape,
then layer on the role-based sections. Read in the listed order.

**Base route by shape:**
- **section-shape / hybrid** → §2 → §10 → §10.5 → §11
- **list-shape** → §2 → §10.5 → §11 — produces many files, so §10 (section generation) does not apply.

**Additive sections (layer onto the base route, in position):**
- **§1 (Session Identity)** — prepend for the code-development family, which mints/reuses a SESSION_ID in output filenames: `researching-code-design`, `planning-code-tasks`, `implementing-code`, `reviewing-code`, `fixing-bugs`, `automation-code-generation`, `automation-test-execution`.
- **§12 (Delegated Exploration)** — insert right after §2 for skills that do broad read-only input sweeps: every `researching-*` skill, plus `reviewing-code`, `planning-code-tasks`, and `fixing-bugs`. Capability-gated and optional (see §12).
- **§13 (Code-Location Discipline)** — apply at the orientation / input-loading step for skills that locate code: every `researching-*` skill, plus `planning-code-tasks`, `implementing-code`, `reviewing-code`, `fixing-bugs`, `conducting-excursion`, and the quick-lane `defining-quick-story` + `implementing-quick-change`. Read `codebase-map.md` before any repository-wide discovery scan (consult-before-scan, not never-scan; see §13). Composes with §12 — map first, then delegate the residual sweep.

**Special routes (override the base route):**
- `implementing-code` → §1 → §2 → §3 (IMPL-STATE schema) → §5 (FIC monitoring) → §7 (if REPAIR) → §10.5 (+ §10.5.3 tool-output retention for build/test/lint runs) → §11
- `fixing-bugs` → §1 → §2 → §12 → §7 (if REPAIR) → §9 (non-interactive) → §10.5 → §11
- `refining-artifact-consistency` (sandbox) → §10.5 only — Stepwise owns `_progress.json` (§2) and apply-back (§11), so both are skipped; read §10.5 + §10.5.1 before any sandbox `Edit`.
- `conducting-excursion` (sandbox / Manual Bay) → §10.5 only — Stepwise owns `_progress.json` (§2), the exit gate, and apply-back (§11), so all are skipped. Multi-turn: read the skill's working-set note (`EXCURSION-NOTES.md`) first on resume turns and do NOT re-run the §2/pre-flight read or re-scan source; read §10.5 + §10.5.1 before any sandbox `Edit`, and apply §10.5.3 (tool-output retention) to every build/test/deploy/probe run.
- `adversarial-platform-review` (discovery / conformance review) → §2 → §10.5 → §11 — list-shape: reads one bounded artifact + its declared target platform, adversarially web-verifies each load-bearing decision, and emits a single cited findings JSON + verdict (no multi-section spec, so §10 does not apply; no broad repo sweep, so §12 does not apply — the web grounding is this agent's own load-bearing work).
- `verifying-artifacts` (verification / checklist + program-verifier) → §2 → §10 → §10.5 → §11 (+ §7 if REPAIR) — section-shape: produces one multi-section VERIFICATION-REPORT (checklist → program_verifiers → judge_scoring → aggregation → verdict). Scores SPEC items + consumed `adversarial-platform-review` findings + a universal anti-gaming item; generates/runs stdlib-only verifier programs for exactly-checkable items (RLCF Fig 6) and mean-of-N judge-samples the rest. No broad repo sweep, so §12 does not apply; no platform re-discovery (that is the discovery layer's job).
- **Catch-all** — any skill not otherwise classified → §2 → §10.5 → §11.

Per-skill specifics that are NOT routing (e.g. MCP/Deterministic-API tooling, manifest-vs-per-file split, approval-token gates) live in each skill's own SKILL.md, not here.

**Universal rules (apply to every skill, regardless of routing):**
- §2 `_progress.json` FIRST ACTION / LAST ACTION write — prevents orchestrator SIGINT. **Exception:** sandbox-mode skills (e.g., `refining-artifact-consistency`, `conducting-excursion`) run inside a Stepwise-managed sandbox where Stepwise owns `_progress.json`; those skills skip §2 and use §10.5 only.
- §10.5 tool discipline (incl. non-ASCII fallback) — applies whenever a skill writes or edits files, NOT only to section-shape skills. The destructive `edit` UTF-8 bug observed 2026-05-19 zeroed out a target file on encoding failure; the §10.5.1 fallback (use full-file replace instead of edit when non-ASCII content is present) is the workaround until the runtime tool is fixed.
- §11 harness output sidecar — last write before `final_response`. Without it the orchestrator records `outputs: {}` and downstream steps cannot pick up declared output parameters. **Exception:** sandbox-mode skills skip §11 because Stepwise owns the apply-back step; see the `refining-artifact-consistency` and `conducting-excursion` special routes above (`§10.5 only`).
- §4 Memory Bank session-end writes (`active-context.md`, `progress.md`) — only when the skill's SKILL.md explicitly opts in; otherwise skip.
- §7 REPAIR bookkeeping — when running in REPAIR mode (a `## ⚠️ Repair feedback` block is present), editing the artifact in place is NOT enough: you MUST also **bump the spec `version`** (semver patch, e.g. `1.0.0` → `1.0.1`) AND **record what changed in the AUDIT file** (set `mode: REPAIR` and append a `## Repair History` entry). Skipping this leaves the version and audit trail stale even though the content was fixed — the documented "iteration applied but version/audit not updated" defect. See §7.2 steps 7–8 for the exact fields and the mandatory self-check.

**REPAIR mode override:** if the prompt contains a `## ⚠️ Repair feedback` block, read §7 (REPAIR Mode) before any other section — even when §7 is not listed in your route. §7 carries the version-bump + audit-delta bookkeeping that is mandatory for every REPAIR (see the universal rule above); the routing rule above intentionally omits it to stay short, so this override is the authoritative trigger.

## Section Index

- §1  — Session Identity (SESSION_ID format, REPAIR reuse rules)
- §2  — `_progress.json` early-signal write (FIRST ACTION / LAST ACTION)
- §3  — IMPL-STATE schema (implementing-code only)
- §4  — Memory Bank cross-session continuity (`active-context.md`, `progress.md`)
- §5  — FIC context monitoring and recovery checkpoint
- §6  — Dual-format input detection
- §7  — REPAIR mode (when `failure_feedback` is provided)
- §8  — Deviation taxonomy (Auto-Fix / Escalate / Defer)
- §9  — Non-interactive execution rules
- §10 — Section-by-section spec generation (skeleton-first + one-section-per-write)
  - §10.1 Section-shape `_progress.json` schema
  - §10.2 Source extraction pass
  - §10.3 Phase A — Skeleton-first
  - §10.4 Phase B — Populate, one section at a time
  - §10.5 Tool discipline (hard rules, includes non-ASCII fallback)
    - §10.5.3 Tool-output retention (context-budget: run full, retain only the verdict / failing slice / distilled evidence)
  - §10.6 CONTINUE on re-entry
- §11 — Harness output sidecar (required when running under Stepwise)
- §12 — Delegated exploration (read-only subagent fan-out, harness-agnostic, optional)
  - §12.1 When to delegate (and when not)
  - §12.2 The return contract (conclusions + `file:line`, never file dumps)
  - §12.3 Model selection (prefer the cheap/fast tier)
  - §12.4 Verify before use (subagent output is input, not ground truth)
- §13 — Code-location discipline (consult `codebase-map.md` before any discovery scan; code-searching skills only)
  - §13.1 The rule — consult before scan, not never scan
  - §13.2 When the map is insufficient — flag the gap
  - §13.3 Applies to (code-searching skills only)
  - §13.4 Composition with §12
  - §12.5 Fallback (no subagent capability → inline exploration)
  - §12.6 Skill-level enforcement

---

## 1. Session Identity

**Format (priority-ordered):**

1. If parameter `feature_id` is set: `{TYPE}-{PROJECT_NAME_UPPER}-{FEATURE_SLUG}-{YYYYMMDD}`
2. Else if the Stepwise session_name is provided by the runtime (capability prompt's `session = '...'` line): `{TYPE}-{PROJECT_NAME_UPPER}-{SESSION_NAME}-{YYYYMMDD}`
3. Else (standalone invocation, no orchestrator, no feature_id): `{TYPE}-{PROJECT_NAME_UPPER}-{YYYYMMDD}`

| Skill | TYPE prefix | Example (feature_id="auth-v2") | Example (session_name="cd-CA-157343") | Example (standalone) |
|-------|------------|--------------------------------|---------------------------------------|----------------------|
| `researching-code-design` | `RESEARCH` | `RESEARCH-ECOMAPP-AUTH-V2-20260411` | `RESEARCH-ECOMAPP-CD-CA-157343-20260411` | `RESEARCH-ECOMAPP-20260411` |
| `planning-code-tasks` | `PLAN` | `PLAN-ECOMAPP-AUTH-V2-20260411` | `PLAN-ECOMAPP-CD-CA-157343-20260411` | `PLAN-ECOMAPP-20260411` |
| `implementing-code` | `IMPL` | `IMPL-ECOMAPP-AUTH-V2-20260411` | `IMPL-ECOMAPP-CD-CA-157343-20260411` | `IMPL-ECOMAPP-20260411` |
| `reviewing-code` | `REVIEW` | `REVIEW-ECOMAPP-AUTH-V2-20260411` | `REVIEW-ECOMAPP-CD-CA-157343-20260411` | `REVIEW-ECOMAPP-20260411` |
| `fixing-bugs` | `QUICKFIX` | `QUICKFIX-ECOMAPP-BUG-002-20260411` | `QUICKFIX-ECOMAPP-QUICK-FIX-20260411` | `QUICKFIX-ECOMAPP-20260411` |

**FEATURE_SLUG derivation (deterministic, no LLM judgement):**
- Lowercase the feature_id, then replace any run of non-alphanumerics with a single hyphen.
- Strip leading/trailing hyphens.
- Truncate to 32 chars.
- Then UPPERCASE the slug for the SESSION_ID (matches the `{PROJECT_NAME_UPPER}` style).
- Examples: `"FEAT LAC Android v2"` → `FEAT-LAC-ANDROID-V2`. `"i18n_pack"` → `I18N-PACK`. `"a/b/c"` → `A-B-C`.

**SESSION_NAME pass-through:**
- The runtime supplies session_name verbatim via the capability prompt; do not re-format. Preserve hyphens and case as given. (e.g., `cd-CA-157343` stays `cd-CA-157343` in the SESSION_ID — uppercased only in the final concatenation when the rest of the ID is uppercased.)

**Rules:**
- Set once at initialization. Never changes within a session.
- REPAIR mode reuses the existing SESSION_ID from the prior run — do NOT generate a new one. To recover the prior SESSION_ID, parse it from the existing spec filename in the input/output folder (preferred) or from the IMPL-STATE header.
  - **In REPAIR, NEVER call `date()`/`today()`/`time()` or recompute the SESSION_ID from parameters.** The date segment is fixed at BUILD time and is immutable across every iteration of the same artifact; minting a fresh date produces a SECOND file (e.g. `…-20260605.md` vs `…-20260610.md`) instead of overwriting the original — the documented duplicate-on-REPAIR defect.
  - **Recovery algorithm:** glob the recorded output folder for the artifact (`<PREFIX>-*.md`); from the matched filename extract the SESSION_ID as the substring between `<PREFIX>-` and `.md`; reuse it verbatim for every write. If multiple match, pick the most recently modified. If a repair target was named but no file matches → see §7 (ABORT, do not BUILD).
- The SESSION_ID propagates verbatim into every artifact filename listed in this protocol (`RESEARCH-SPEC-{SESSION_ID}.md`, `PLAN-SPEC-{SESSION_ID}.md`, `IMPL-STATE-{SESSION_ID}.md`, `REVIEW-SPEC-{SESSION_ID}.md`, etc.). Filenames are the primary collision-prevention surface — do NOT shorten or omit the SESSION_ID in any output filename.

**Why this format:** Prior runs that used `{TYPE}-{PROJECT}-{YYYYMMDD}` collided when two sessions ran on the same project on the same day, silently overwriting each other's specs. Including feature_id (or session_name as a fallback) keeps every concurrent or sequential run uniquely addressable in the same project folder.

---

## 2. _progress.json — Early Signal Write

**Purpose:** Prevents the orchestrator coordinator from sending SIGINT to a running skill.

Write this as the **FIRST file action** in any skill execution (before creating any other outputs):

```json
{
  "skill": "{skill-name}",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "started_at": "{ISO timestamp}",
  "completed_at": null,
  "total": 0,
  "completed": 0,
  "items": []
}
```

Write to: `{output_folder}/_progress.json`

**Update cadence:**
- On FIC trigger (Section 5): `"status": "COMPACTION_NEEDED"`
- On completion: `"status": "COMPLETED"`, `"completed_at": "{ISO timestamp}"`
- On failure: `"status": "FAILED"`, `"completed_at": "{ISO timestamp}"`

**Schema variants.** A skill picks ONE shape based on its work model. The common
envelope (`skill`, `session_id`, `status`, `started_at`, `completed_at`) is identical
across variants.

- **List shape (default — shown above)** — uses `total`, `completed`, `items[]`. Suits
  skills that produce many discrete artifacts (e.g. `implementing-code` writing source
  files; `planning-code-tasks` tracking task entries).
- **Section shape** — uses `skeleton_written` (bool) + `sections{}` map. Suits skills
  that produce a single multi-section spec file from large unstructured inputs. See
  **Section 10** for the full protocol (skeleton-first + one-section-per-write +
  source extraction + CONTINUE re-entry).

### Skill-Level Enforcement (FIRST ACTION / LAST ACTION)

Every skill MUST include these two inline callouts — agents do not reliably follow
protocol-only references for session bookend writes:

**At session start** (after folder creation, before any other file write):

> **FIRST ACTION — MANDATORY:** Write `_progress.json` to the output folder before any other file write.
> This prevents the orchestrator from sending SIGINT.

Include the minimal JSON schema inline in the skill so the agent can write immediately
without reading this file first. Use `"session_id": "initializing"` as placeholder.

**At session end** (after all spec files written, after Memory Bank writes):

> **LAST ACTION — MANDATORY:** Update `_progress.json` status to `COMPLETED` with `completed_at` timestamp.
> If the session failed, set status to `FAILED` instead.

These callouts are NOT infrastructure duplication — they are the minimum viable bridge
between protocol and agent behavior. The full lifecycle lives here; the callouts trigger it.

**For `implementing-code` only** — also include `output_contract` (survives compaction):

```json
{
  "skill": "implementing-code",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "output_contract": {
    "progress_folder_path": "{progress_folder_path}",
    "impl_state_file": "IMPL-STATE-{session_id}.md",
    "source_path": "{source_path}",
    "expected_outputs": ["source_path", "impl_state_path"]
  }
}
```

Pin the same values in the IMPL-STATE header under `output_contract`. After context
compaction, read IMPL-STATE header (not filesystem) to recover the output path and format.

---

## 3. IMPL-STATE Schema (implementing-code only)

Single consolidated tracking file. Written at Phase A as skeleton tables (header + separator
rows only — no example data). Rows appended after each file write using EOL append mode —
never rewrite or replace the whole file. Use Edit tool to insert at end of table.

```
IMPL_INDEX = {
  session_id: string,
  project_name: string,
  source_path: string,
  project_root: string,             // same as source_path
  mode: "STANDARD" | "REPAIR",
  input_format: "AGENT_NATIVE" | "LEGACY",
  active_phase: string,
  output_contract: {
    progress_folder_path: string,
    impl_state_file: string,
    source_path: string,
    expected_outputs: [string]
  },
  phases: [{ id, status, started_at, completed_at, files_count, tests_count }],
  files_touched: [{ phase, path, action, status, test_file }],
  repair_log: [{ iteration, phase, feedback, files_modified: [{ path, change_summary }] }],
  blockers: [{ phase, description, timestamp, resolution }],
  deviations: [{ phase, description, reason, impact }],
  build_commands_discovered: [],
  tech_stack_detected: {},
  tool_results: {},
  execution_log: [],
  metrics: {},
  validations: {},
  open_questions: []
}
```

**BLOCKING gate:** Do NOT generate the next file until the files_touched append has been
confirmed as a completed tool call. Batching updates = lost progress on interruption.

### 3.1 IMPL-STATE Write Budget (anti-heartbeat-churn)

IMPL-STATE writes are limited to:
  (a) skeleton write at Phase A start,
  (b) one append per file completed in Phase B (already required by the §3 BLOCKING gate),
  (c) one write at each phase boundary,
  (d) one final write at end.

Total writes per run MUST satisfy:

    writes_to_IMPL_STATE  <=  files_touched + phases + 2

All Phase B writes MUST be APPEND-MODE row inserts into `files_touched`.
Full-file rewrites of IMPL-STATE between sub-phases are FORBIDDEN.

Self-check before any IMPL-STATE write:

    heartbeat_ratio = existing_IMPL_STATE_writes / source_or_test_files_written_so_far
    IF heartbeat_ratio > 2.0 → skip this write. Resume only after the next
    source/test file is written.

Rationale: every IMPL-STATE write consumes one agent iteration. Excess
heartbeats are the leading cause of iteration-cap exhaustion before the
final sub-phase of medium-sized implementations.

---

## 4. Memory Bank — Cross-Session Continuity

Two files in `./context-pack/` provide every session's agent with immediate context
about what happened before — without re-reading all prior session artifacts.

```
context-pack/
├── active-context.md    ← "What is happening NOW?" (overwritten each session)
└── progress.md          ← "What HAS been accomplished?" (append-only ledger)
```

Both live at the **project root** `./context-pack/` — shared across all capabilities
so `progress.md` becomes a unified cross-capability timeline.

### RULE 10 — active-context.md (Session Lifecycle)

At SESSION START: write `context-pack/active-context.md` with current session ID,
scope, and any prior session context loaded above. Status: `STARTING`.
At SESSION END: overwrite with final state (status, decisions, blockers, key artifacts).
If the session crashes before finalization, the SESSION START write provides
"where we were" context for the next session.

### RULE 11 — progress.md (Per-Phase Milestone Ledger)

After EVERY phase completes (NOT just the final phase): append one milestone row to
`context-pack/progress.md` (append-only — never overwrite, never delete prior rows).
Writing IMPL-STATE does NOT satisfy this write — they are different files with different
scopes. Skipping an intermediate phase write breaks per-phase crash recovery.

### Session Start

```
CONTEXT_PACK_PATH = ./context-pack

IF exists({CONTEXT_PACK_PATH}/active-context.md):
  READ active-context.md
  EXTRACT: prior_session_id, status, current_focus, open_blockers, decisions_log
  LOG: "Memory Bank: loaded active-context from session {prior_session_id}"
  # Use to: avoid re-discovering known blockers, continue from prior state,
  #         respect decisions already made (do not contradict them)
ELSE:
  LOG: "Memory Bank: No active-context.md found. First session."

IF exists({CONTEXT_PACK_PATH}/progress.md):
  READ progress.md
  EXTRACT: milestone_count, last_milestone
ELSE:
  LOG: "Memory Bank: No progress.md found. First session."

# Write active-context.md at session START (crash recovery — mandatory)
WRITE {CONTEXT_PACK_PATH}/active-context.md:
  ---
  document_type: active-context
  session_id: {SESSION_ID}
  capability: {capability_name}
  skill: {skill_name}
  project: {project_name}
  last_updated: {ISO timestamp}
  status: STARTING
  ---
  # Active Context: {project_name}
  ## Current Focus
  Session {SESSION_ID} initializing. Skill: {skill_name}. Mode: {BUILD | REPAIR}.
  ## Prior Session Context
  {IF prior loaded: Previous: {prior_session_id} — status: {prior_status}. Include blockers/decisions.}
  {ELSE: First session — no prior context.}
  ## Open Blockers
  {Copy from prior session if any, else "None."}
```

### Session End

```
# 1. Overwrite active-context.md with final state
WRITE {CONTEXT_PACK_PATH}/active-context.md:
  ---
  status: {COMPLETED | FAILED | BLOCKED}
  session_id: {SESSION_ID}
  last_updated: {ISO timestamp}
  ---
  # Active Context: {project_name}
  ## Current Focus
  {COMPLETED: "{skill_name} complete. Produced: {artifact summary with counts}."}
  {FAILED: "{skill_name} failed. Reason: {description}."}
  {BLOCKED: "{skill_name} blocked. Blocker: {description}. Required: {resolution}."}
  ## Decisions Log
  Append-only within a session. Preserve ALL rows from prior sessions.
  | ID | Decision | Rationale | Impact | Reversible | Session |
  |----|----------|-----------|--------|------------|---------|
  {Carry forward all prior rows. Append new decisions from this session.}
  {ID format: D-{NNN}, sequential across sessions, never reset.}
  {If no new decisions: keep prior rows, add nothing.}
  ## Open Blockers
  {Unresolved issues, or "No open blockers."}
  ## Key Artifacts
  | Artifact | Path | Status |
  |----------|------|--------|
  | {primary output} | {path} | written |

# 2. Append milestone to progress.md
IF NOT exists({CONTEXT_PACK_PATH}/progress.md):
  CREATE with header:
    ---
    document_type: progress-ledger
    project: {project_name}
    created_at: {ISO timestamp}
    ---
    # Progress Ledger: {project_name}
    | Session | Date | Capability | Skill | Status | Artifacts | Notes |
    |---------|------|------------|-------|--------|-----------|-------|

APPEND one row:
  | {SESSION_ID} | {date} | {capability} | {skill_name} | {STATUS} | {N <artifact-type>} | {1-line note} |
  # {N <artifact-type>} MUST use the specific type from the registry below.
  # Examples: "12 tasks (4 phases)", "48 files, 32 tests", "17 review-findings"
  # NEVER write: "1 artifact", "N artifacts", or leave blank.
```

Both writes are **MANDATORY** — even on failure, record the failure state.

### Verification After Writing progress.md

```
VERIFY exists(context-pack/progress.md)
IF NOT exists: LOG "ERROR: write FAILED. Retrying." → RETRY (mandatory tool call)
IF still fails: LOG "CRITICAL: Memory Bank write failed. Manual intervention required."

READ last line — VERIFY it contains SESSION_ID AND specific artifact count (not "N artifacts")
IF generic: REWRITE the row with the correct count from the registry.

LOG: "Memory Bank: progress.md verified — {N} rows. Last: {SESSION_ID} | {artifact_count}"
```

This verification is **NON-NEGOTIABLE**.

### Artifact Type Registry

| Skill | Artifact Type | Example |
|-------|--------------|---------|
| `researching-adrs` | ADRs | `12 ADRs` |
| `researching-bounded-contexts` | bounded-contexts | `7 bounded-contexts` |
| `establishing-architecture-foundation` | services | `9 services` |
| `specifying-architecture` | architecture-docs | `3 architecture-docs` |
| `researching-prd` | requirements | `24 requirements` |
| `planning-epics` | epics | `15 epics` |
| `implementing-user-stories` | user-stories | `48 user-stories` |
| `researching-code-design` | research-docs | `20 research-docs` |
| `planning-code-tasks` | tasks | `12 tasks (4 phases)` |
| `implementing-code` | source-files + tests | `48 files, 32 tests` |
| `reviewing-code` | review-findings | `17 review-findings` |
| `adversarial-platform-review` | conformance-findings | `4 conformance-findings (B1-B4 BLOCKER)` |
| `generating-test-cases` | test-cases | `35 test-cases` |
| `generating-e2e-test-cases` | E2E-test-cases | `22 E2E-test-cases` |
| `creating-qe-master-plan` | test-domains | `8 test-domains` |
| `benchmarking-execution` | benchmark-results | `1 benchmark-results` |
| `db-migration-quality-planning` | migration-test-domains | `5 migration-test-domains` |
| `db-parity-execution` | parity-diff-tables | `180 parity-diff-tables` |
| `db-source-profiling` | profiled-tables | `210 profiled-tables` |
| `defining-qe-strategy` | strategy-sections | `6 strategy-sections` |
| `formatting-platform-export` | work-items | `63 work-items` |
| `secreview-intake-validation` | validated-intake-package | `1 validated_intake_package` |
| `calibrating-catalog` | catalog-entries | `14 catalog-entries` |
| `creating-test-master-plan` | test-plan-sections | `10 test-plan-sections` |
| `analyzing-impact` | impacted-tests | `18 impacted-tests` |
| `developing-test-scripts` | test-scripts | `12 test-scripts` |
| `executing-automation` | executed-tests | `25 executed-tests` |
| `secreview-context-correlation` | service-graph-elements | `14 service-graph-elements` |
| `secreview-finding-enrichment` | validated-findings | `10 validated_finding records, P1–P4 register` |
| `secreview-report-generation` | report-pdfs | `2 report PDFs (technical + executive); case Closed` |
| `executing-tests` | executed-tests | `30 executed-tests` |
| `scoping-engagement-baseline` | scope-baseline-rows | `18 baseline rows across 4 capabilities` |
| `estimating-supervision` | supervision-plan | `4.5 FTE across 5 roles, 6 months (pre_screening)` |
| `assembling-estimation-proposal` | proposal + internal-audit | `1 proposal + 1 internal-audit (margin 2.4x)` |
| `interviewing-code` | knowledge-graphs | `1 knowledge-graphs` |
| `validating-code-interview` | analysis-reports | `1 analysis-reports` |
| `designing-structure` | documentation-outlines | `1 documentation-outlines` |
| `discovering-kit-templates` | template-candidates | `5 template-candidates` |
| `scaffolding-kit-extensions` | resolved-extensions | `14 resolved-extensions` |
| `secreview-static-scanning` | consolidated-findings | `42 consolidated-findings across 3 repos` |
| `generating-omniverse-boilerplate` | source-files | `9 files, 3 tests` |
| `generating-content` | draft-documentation | `1 draft-documentation` |
| `refining-iteratively` | refined-documentation | `1 refined-documentation` |
| `reverse-engineering-product` | discovery-deliverables | `11 discovery-deliverables (FRD, intent, flows, data-model, tokens-raw, tokens-coverage, audit, selectors, text-inventory, behaviors-catalog, figma-tokens.json)` |
| `coverage-gap-analyzer` | coverage-reports | `2 coverage-reports (gap-report-{date}-{project}.md, recommended-tcs-{date}-{project}.md)` |
| `coverage-judge` | coverage-reports | `1 coverage-reports (final-coverage-report-{date}-{project}.md)` |
| `pentest-intake-normalization` | engagement-intake | `1 engagement-intake (10 sections)` |
| `pentest-scope-validation` | authorized-scope-register | `1 authorized-scope-register` |
| `pentest-asset-verification` | asset-verification-evidence | `1 asset-verification-evidence` |
| `pentest-roe-assembly` | roe-package | `1 roe-package (7-gate checklist)` |
| `pentest-recon-planning` | recon-plan | `1 recon-plan` |
| `pentest-surface-enumeration` | attack-surface-findings | `1 attack-surface-map + N findings` |
| `pentest-technology-fingerprinting` | technology-fingerprint | `1 technology-fingerprint` |
| `pentest-scope-boundary-flagging` | public-exposure-findings | `N public-exposure-findings` |
| `pentest-webapi-test-planning` | webapi-test-plan | `1 webapi-test-plan` |
| `pentest-auth-session-testing` | auth-session-findings | `N auth-session-findings` |
| `pentest-owasp-web-testing` | web-findings | `N web-findings` |
| `pentest-owasp-api-testing` | api-findings | `N api-findings` |
| `pentest-ai-asset-mapping` | ai-asset-map | `1 ai-asset-map` |
| `pentest-prompt-injection-testing` | prompt-injection-findings | `N prompt-injection-findings` |
| `pentest-rag-memory-testing` | rag-memory-findings | `N rag-memory-findings` |
| `pentest-tool-mcp-testing` | tool-mcp-findings | `N tool-mcp-findings` |
| `pentest-ai-resource-exhaustion-testing` | ai-resource-findings | `N ai-resource-findings` |
| `pentest-mobile-scoping` | mobile-asset-map | `1 mobile-asset-map` |
| `pentest-mobile-static-analysis` | binary-analysis-findings | `N binary-analysis-findings` |
| `pentest-mobile-dynamic-analysis` | mobile-runtime-findings | `N mobile-runtime-findings` |
| `pentest-mobile-auth-transport-testing` | platform-misconfig-findings | `N platform-misconfig-findings` |
| `pentest-infra-enumeration` | network-service-inventory | `1 network-service-inventory` |
| `pentest-infra-misconfig-testing` | infrastructure-exposure-findings | `N infrastructure-exposure-findings` |
| `pentest-infra-impact-summary` | perimeter-risk-summary | `1 perimeter-risk-summary` |
| `pentest-poc-planning` | poc-plan | `1 poc-plan` |
| `pentest-poc-execution` | validated-poc-evidence | `N validated-poc-evidence` |
| `pentest-exploit-chain-analysis` | exploit-chain-analysis | `N exploit-chain-analysis` |
| `pentest-post-exploitation` | post-exploitation-findings | `N post-exploitation-findings` |
| `pentest-blast-radius-assessment` | blast-radius-assessment | `1 blast-radius-assessment` |
| `pentest-poc-evidence-packaging` | signed-poc-package | `1 signed-poc-package + retest-scripts` |
| `pentest-finding-consolidation` | consolidated-findings | `1 consolidated-findings` |
| `pentest-severity-taxonomy-mapping` | cvss-cwe-owasp-mapping | `N cvss-cwe-owasp-mapping` |
| `pentest-compliance-mapping` | compliance-control-matrix | `1 compliance-control-matrix` |
| `pentest-report-generation` | security-report | `2 reports (executive, technical)` |
| `pentest-evidence-signing` | signed-evidence-package | `1 signed-evidence-package` |
| `pentest-remediation-prioritization` | remediation-roadmap | `1 remediation-roadmap` |
| `pentest-remediation-guidance` | remediation-guidance | `N remediation-guidance` |
| `pentest-remediation-register` | remediation-status-register | `1 remediation-status-register` |
| `pentest-retest-validation` | retest-results | `N retest-results` |
| `secreview-memory-synthesis` | scan-memory-entries | `N memory entries` |
| `verifying-artifacts` | verification-reports | `1 verification-reports` |

If your skill is not listed: count the PRIMARY deliverable artifacts and use a descriptive plural type.

### Adding a New Skill to the Registry

When creating a new skill (via `engineering-skills` or manually), add a row to this table:
1. Skill name (kebab-case, matches SKILL.md folder name)
2. Artifact type (descriptive plural noun)
3. Example count string

Also update the skill's session-end block with the matching artifact type one-liner.

---

## 5. FIC Context Monitoring + Recovery Checkpoint

Monitor context utilization throughout execution.

**On hitting 60% context threshold:**
1. IMMEDIATELY write current output state to disk (even if incomplete)
2. Mark incomplete sections: `status: pending — context compaction triggered`
3. Update `_progress.json`: `{ "status": "COMPACTION_NEEDED" }`
4. WRITE Recovery Checkpoint (see format below)
5. LOG: `"FIC ALERT: Context at {N}% — partial state written for session recovery"`

**Recovery Checkpoint** — write to `{progress_folder_path}/RECOVERY-CHECKPOINT-{session_id}.md`:

```
(a) Current phase + current file being worked on
(b) Files COMPLETED in this session (paths only — do NOT re-read them)
(c) Key patterns/decisions from plan and research (compact summary, max 30 lines)
(d) Output file contract: path to _progress.json + expected output parameters
(e) Next action to take when resuming
```

Note: IMPL-STATE tracks status metadata. Recovery Checkpoint tracks WORKING CONTEXT —
what was in memory when compaction hit. They serve different purposes.

**After context continuation (compaction boundary), FIRST actions:**
1. READ Recovery Checkpoint
2. READ IMPL-STATE
3. Resume from the "next action" in the checkpoint
4. Do NOT re-read source files already marked COMPLETED in IMPL-STATE files_touched
5. Re-read ONLY the current file being worked on + plan/research sections for remaining work

---

## 6. Dual-Format Input Detection

All skills support two input formats. Detection is automatic.

| Format | Signal | Handling |
|--------|--------|---------|
| Agent-native | Single `*-SPEC-*.md` in input folder | Parse structured sections directly |
| Legacy | Multiple numbered files (00-XX) in input folder | Read index (00-*), then load files on demand |

Agent-native is preferred. Legacy is for backward compatibility.

---

## 7. REPAIR Mode

Triggered when `failure_feedback` parameter is provided.

### 7.1 FIRST ACTION — Mandatory Reviewer-Comment Absorption

Before any spec write in REPAIR mode, the agent MUST:

```
1. Read the latest state.json for the current capability/session.
2. Extract every navigation_history[] entry where action == "failed_step"
   AND step matches the current step being repaired.
3. Concatenate the `reason` text of those entries in timestamp order.
   This is the AUTHORITATIVE reviewer-comment payload — it overrides any
   abbreviated version that may have been passed via the failure_feedback
   parameter alone.
4. Scan that text for INLINE FILE REFERENCES of the form:
     - "Line {N} (near: …)"           — line-anchored content correction
     - "File: {path}"                  — file-anchored content correction
     - "comments file …: {path}"       — separate comment store
   If a separate comment store path is mentioned, load that file and treat
   its contents as additional ground-truth corrections.
5. Quote every reviewer instruction verbatim in the agent's internal
   reasoning before proposing any regeneration. Do NOT paraphrase reviewer
   text in the spec — copy the resolution faithfully.
```

Skipping any of steps 1-5 is a protocol violation. The reviewer-comment payload
loaded here feeds into Section 7.2 (the directive parser).

### 7.2 Directive Application

```
1. Load existing output spec — do NOT start from scratch.
2. If SESSION_ID needed: extract from existing spec filename.
3. Parse the reviewer-comment payload from §7.1 into directives:
   { section: "section_name" | "global", instruction, reason }
4. FOR EACH section in spec:
   - Has directive → load from existing spec, apply change, rewrite section
   - No directive → preserve existing content verbatim. Do NOT touch it.
5. "global" directive → re-execute full generation (Steps 3 onward)
6. Re-run all validation checks after fixes.
7. Produce repair_delta: { fixed: [], still_failing: [], regressions: [] }
8. Version bump + audit trail (MANDATORY — do NOT skip after editing content):
   a. Read the current `version` from the existing spec's front-matter.
   b. Write `version = increment_patch(current)` (e.g. 1.0.0 → 1.0.1) back into the
      SPEC front-matter. Reuse the same filename (§1 — never re-date the filename).
   c. In the AUDIT file (same SESSION_ID): set `mode: REPAIR`, set its `version` to the
      new value, and APPEND (do not overwrite) a `## Repair History` entry:
        - `version`, `timestamp`
        - `directives_applied`: the reviewer directives you actioned (verbatim)
        - `sections_changed`: section names you rewrote
        - `sections_preserved`: section names left untouched
        - `repair_delta`: { fixed, still_failing, regressions } from step 7
9. SELF-CHECK before `final_response` (MANDATORY): confirm (a) the spec `version` you
   wrote is strictly greater than the version you read in 8a, and (b) the AUDIT file
   contains the new `## Repair History` entry. If either is false, you edited content
   but skipped the bookkeeping — fix it now, before finishing. Editing the artifact in
   place WITHOUT bumping the version and appending the audit entry is a §7 violation.
```

REPAIR does NOT start over. Sections without directives are unchanged.

**Path Consistency Rule:** REPAIR output MUST be written to the same folder AND the same
filename as the original run. Read the original output path from the existing spec file,
IMPL-STATE, or `_progress.json` — do NOT re-derive it from parameters. If the original run
used a `feature_id`-scoped path (e.g., `{project}/i18n/code-impl-output/`), the REPAIR run
writes to that same path. Creating a new folder at a different level fragments the artifact
tree and breaks downstream step references.

**Discovery + reuse algorithm (mandatory):**
```
1. Determine the REPAIR search directory:
   - If failure_feedback names a rejected artifact path (e.g. "review rejected for: <path>"),
     use dirname(<path>) as the search dir and basename(<path>) as the target filename.
     Do NOT use the (possibly re-rendered) output-path parameter for discovery.
   - Otherwise use the recorded output path (existing spec / IMPL-STATE / _progress.json).
2. Glob that directory for the artifact (e.g. `<PREFIX>-*.md`). Reuse the matched file's
   dirname AND basename VERBATIM for the write — never recompute SESSION_ID/date (see §1).
3. ABORT, do NOT BUILD: if a rejected path was named but no matching file is found, ABORT
   with guidance — e.g. "REPAIR target not found: <path>. Refusing to BUILD a fresh spec,
   which would duplicate the artifact. Verify the output path or restore the rejected file."
   Silently falling back to a fresh BUILD on a divergent path is the documented duplicate defect.
```

**Violation Signal:** If, during REPAIR, the regenerated spec still contains content
that the reviewer directive explicitly rejected (e.g., still says "greenfield" after
the reviewer rejected greenfield wording), this is a §7 violation. Abort the write,
re-run §7.1, and try again.

---

## 8. Deviation Taxonomy

When implementation encounters something not in the plan, classify and act:

### Auto-Fix (proceed without stopping)
- **D-AUTO-1: Bug in generated code** — syntax error, runtime error, failing test caused by current task
- **D-AUTO-2: Missing critical safety** — null check, error handling, input validation, auth guard omitted
- **D-AUTO-3: Broken dependency** — missing import, unresolved reference, config error blocking build

Rules: Max 3 auto-fixes per file. Document each in IMPL-STATE deviations table.
If 4th auto-fix needed on the same file → escalate.

### Escalate (stop, document, request guidance)
- **D-ESC-1: Architectural change** — new database table, new service, schema migration, library substitution
- **D-ESC-2: Scope expansion** — feature not in plan, new API endpoint, new UI component
- **D-ESC-3: Spec contradiction** — plan says X, research says Y, code needs Z

Action: Write blocker to IMPL-STATE, set phase status = BLOCKED, include alternatives
with trade-offs. Do NOT proceed without guidance.

### Defer (note for future, continue current work)
- **D-DEF-1: Pre-existing issue** — bug/smell in untouched code discovered during implementation
- **D-DEF-2: Optimization opportunity** — performance improvement not in scope
- **D-DEF-3: Style/convention mismatch** — naming inconsistency in existing code

Action: Add to IMPL-STATE deviations with category = DEFERRED. Do NOT fix.

---

## 9. Non-Interactive Execution

When invoked by an automated capability step, this skill runs in non-interactive mode:
- Do NOT ask clarifying questions or pause for human input
- Make reasonable assumptions based on the provided parameters
- Document any assumptions in the output under `## Assumptions` or in `open_questions`
- If blocked by missing required information, write a Gap Report and EXIT — do not improvise

---

## 10. Section-by-Section Spec Generation

**Applicability.** Skills that produce a single multi-section spec file from large unstructured inputs (project briefs, transcripts, legacy docs, prior research artifacts). Examples: `researching-prd`, `planning-code-tasks`, `implementing-user-stories`, `generating-test-cases`, `defining-qe-strategy`, `researching-feature-impl`.

Skills that produce many small files (e.g. `implementing-code` writing source files) use the list-shape `_progress.json` variant (Section 2) — this section does not apply.

**Failure modes prevented.**
- **Silent SIGINT** from long thinking loops with no on-disk progress (skeleton-first fixes this).
- **Gateway 504** from generating all sections in a single huge completion (one-section-per-write fixes this).
- **Input-token explosion** from re-reading raw sources on every section (extraction pass fixes this).
- **Bulk-rewrite fallback** when targeted edits hit a snag (Edit-only discipline + ASCII stub text fix this — see Tool Discipline below).

---

### 10.1 Section-shape `_progress.json`

```json
{
  "skill": "{skill-name}",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "started_at": "{ISO timestamp}",
  "completed_at": null,
  "skeleton_written": false,
  "sections": {
    "{section_name_1}": "pending",
    "{section_name_2}": "pending"
  }
}
```

Per-section status flips `pending` → `complete` after the section is populated and the `_progress.json` update has been written. `skeleton_written` flips to `true` after Phase A completes.

---

### 10.2 Source Extraction Pass

**Trigger.** Run extraction if ANY of:
- More than 2 source files, OR
- Any single source file > 200 lines, OR
- Cumulative source content > 500 lines.

If none of the above (small bounded input), SKIP — read sources directly in Phase B.

**Mechanism.**

```
mkdir -p {SPEC_FOLDER}/_extractions/

FOR EACH source in inputs:
  extraction_path = {SPEC_FOLDER}/_extractions/{basename(source)}.md

  IF MODE == REPAIR AND extraction_path exists AND source unchanged since prior run:
    SKIP — reuse existing extraction.
    LOG to SOURCE_LOG: "reused extraction: {extraction_path}"
    CONTINUE.

  Read source.
  Distill into extraction_path as a structured bullet list, using the
  extraction schema declared by the skill (each SKILL.md defines its own
  schema — fields differ by domain).
  Preserve source line numbers on every entry for traceability.

  Write extraction_path.
  LOG to SOURCE_LOG: { source: {raw_path}, extraction: {extraction_path} }
```

After the pass:
- Inputs reference extraction files, NOT raw source paths.
- Phase B reads extractions only. Raw sources are dropped from working memory.
- The audit must list extraction files alongside raw sources so provenance is intact: `raw_source → extraction → spec_item`.

**Zero Invention still wins.** Extraction is distillation, not invention. Facts not in the source do not appear in the extraction. Inferred industry-standard hints remain `[INFERRED]` in the extraction and become assumption entries (with appropriate IDs) in Phase B.

---

### 10.3 Phase A — Skeleton-First (within 5 tool calls of entering generation)

1. Write SPEC_FILE skeleton: header metadata + section stubs. Each stub is exactly: section heading + `status: pending - will be generated`. **Use ASCII hyphen only** (U+002D) — never em dash, never en dash. Non-ASCII characters in the stub create downstream encoding traps for any agent that later reaches for shell tooling.
2. Update `_progress.json`: set `skeleton_written: true`. All `sections` remain `pending`.

If you find yourself in a 3rd `think` block before the skeleton write, STOP and write the skeleton now. Count ANY tool call (think, view, bash, edit).

**Tool:** `Write` for the initial skeleton (single file create).

---

### 10.4 Phase B — Populate, One Section At A Time

For each section, in the order defined by the skill's template:

1. **Gather only what this section needs:**
   - Read the relevant `_extractions/*.md` entries (or raw sources if extraction was skipped per 10.2 thresholds).
   - Read SPEC_FILE ONLY IF this section depends on cross-section state. Each SKILL.md declares its cross-section dependency list.
2. Generate the section body using only the inputs gathered in step 1.
3. **Replace the section's stub with populated content using `Edit`.** One `Edit` call, one section.
4. Update `_progress.json`: set `sections.{section_name}: complete`.
5. **Drop the section body from working memory before moving to the next section.** Do not carry populated section content forward in your reasoning. If a later section needs it (per the skill's dependency list), re-read SPEC_FILE then.

---

### 10.4.1 Per-Section Context Budget (hard cap)

**Why this exists.** A REPAIR/CONTINUE run once hit ~2M prompt tokens with **zero sections written** before a 504 gateway timeout: it preloaded every `_extractions/*.md` file upfront and planned all pending sections in a single `think` block, so Phase B never started. The per-section budget below makes that failure mode unrepresentable.

**Hard rules — applied at the start of EACH section iteration in Phase B:**

1. **Load only what THIS section needs.** Before generating section `N`:
   - You may have loaded in this iteration: `_progress.json`, the SPEC_FILE (only if Section 10.4 step 1 requires it for cross-section state), and the `_extractions/*.md` files that contain entries for section `N`.
   - You may NOT have loaded: extraction files that feed only other sections, raw source files (those were distilled in Step 10.2 and must not be re-read in Phase B — already a §10.5 rule, restated here as a budget concern), prior populated section bodies still in your reasoning trace from earlier iterations.

2. **No multi-section planning.** Do NOT compose a `think` block that drafts content for more than one pending section. One section per inference call applies to **reasoning**, not just to writes. A `think` block that outlines sections `N`, `N+1`, `N+2` is a §10.4.1 violation — split it into one micro-plan immediately before section `N`'s `Edit` call.

3. **Drop between sections.** After section `N`'s `Edit` succeeds and `_progress.json` is updated:
   - Drop section `N`'s body, the extraction files you read for it, and the micro-plan from your reasoning trace.
   - The next iteration starts from disk truth (`_progress.json` + targeted re-reads), not from accumulated working memory.

4. **Soft prompt-size gate.** If you can observe your prompt size, treat **150K tokens** as the per-iteration soft cap. If you reach it before writing section `N`:
   - Stop loading. Write section `N` with what you have, or mark fields you cannot fill as `pending` + register them in `open_questions`.
   - Do NOT keep reading "one more extraction to be thorough." That is the pattern that produces the 2M-token failure.

5. **Symptom of violation.** If you find yourself in your 5th `view` call of Phase B without an intervening `Edit` to SPEC_FILE, you are violating §10.4.1. STOP, write the current section with the extractions already loaded, then resume.

**Why a budget and not a planning step:** A planning step ("first decide which extractions feed which section, then read them") is itself a load. The cheapest enforceable rule is "don't have loaded what you don't need RIGHT NOW" — checked at section-iteration boundaries, not via upfront planning.

---

### 10.5 Tool Discipline (Hard Rules)

These rules exist because each one has been observed as a failure mode. Violations defeat the protocol even when the prose discipline appears to be followed.

- **Skeleton write: use `Write`.** Single tool call, single file creation. Do NOT compose the skeleton via `Bash` heredoc.
- **Section population: use `Edit`.** Targeted find-and-replace, one section per call. The `Edit` tool handles unicode natively and validates the prior content matches what you expect.
- **NEVER use `Bash` + `sed` / `awk` / `python3 <<EOF` / `cat >` to mutate SPEC_FILE.** These are bulk-rewrite paths disguised as targeted operations. They read the whole file into memory, transform it, and write it back — the exact pattern Phase B is designed to prevent. They also bypass `Edit`'s prior-content validation, so a bug in the script silently corrupts the spec.
- **If `Edit` fails on a section, fix the `Edit` call.** Provide more surrounding context to make `old_string` unique. Do NOT switch to `Bash` / `python3` as a workaround — that path has been observed to cascade: agent hits one snag, then rewrites all remaining sections in a single script, violating one-section-per-write.
- **NEVER compose multiple sections in memory before writing.**
- **NEVER emit more than one section per Write/Edit call.**
- **NEVER re-read raw sources during Phase B.** Read `_extractions/*.md` files, or SPEC_FILE for declared cross-section state.
- **Prefer ASCII punctuation in authored content.** Emit ASCII equivalents for *decorative* non-ASCII glyphs the model tends to insert: `-` or ` - ` for em/en dashes (`—`/`–`), `->` for `→`, straight quotes `"` / `'` for smart quotes (`“ ” ‘ ’`), `...` for `…`, `-`/`*` for `•`, `x` for `×`, plain space for non-breaking space. This keeps English-output specs on the fast, safe targeted-`Edit` path by avoiding the §10.5.1 non-ASCII fallback (which forces full-file replace). **This rule removes gratuitous glyphs only — it NEVER changes the output language or strips *required* non-ASCII.** Accented and localized characters in non-English output (`á é í ó ú ñ ü ç ã …`) are content, not decoration: preserve them exactly and let §10.5.1 handle the run. Do not transliterate them to "fix" the fallback.

If a section's content genuinely will not fit a single `Edit` call (extreme size), split that one section into sub-edits — but every sub-edit is still an `Edit` call against SPEC_FILE, never a Bash mutation.

---

#### 10.5.1 Non-ASCII Content Fallback (TEMPORARY — runtime workaround)

**Status:** TEMPORARY. Remove this fallback once the runtime's targeted-edit operation supports UTF-8 reliably.

**Why this exists.** Some runtimes' targeted-edit operations have been observed to fail with `'ascii' codec can't encode character ...` errors when the `new_string` contains non-ASCII codepoints (accented vowels in Spanish / French / Portuguese / German, em dashes, smart quotes, etc.). The same failure mode has been observed to **destroy the target file** (the tool truncates the file before catching the encode exception, so a failed edit zeroes out the prior content). Until the runtime is fixed, targeted-edit cannot be trusted for SPEC_FILEs that may contain non-ASCII text.

**Detection (at Phase B entry).** If ANY of the following is true, apply this fallback for **every** section write in the run:
- The detected output language (per Step 2) is not English.
- Any input source or `_extractions/*.md` file contains a character with codepoint > 127.
- Detection is unreliable / ambiguous — default to this fallback. The cost is bounded.

**Per-section flow under fallback:**

1. **Read** SPEC_FILE from disk — this is the current state (skeleton + sections populated so far).
2. **Generate** this section's body in working memory using the inputs from Phase B step 1.
3. **Compose** the full file payload in working memory:
   - header / metadata block (verbatim from disk)
   - all previously populated sections (verbatim from disk)
   - this section, freshly populated
   - remaining stubs (verbatim from disk)
4. **Write** the composed payload via the full-file replace operation (`Write` in Claude Code, `replace` in other harnesses). One call, full file content.
5. **Update** `_progress.json`: set `sections.{section_name}: complete`.
6. **Drop** everything from working memory before the next section. SPEC_FILE on disk is the source of truth; do not carry composed content forward.

The per-section discipline is unchanged: one section populated per inference call, working memory bounded, prior section bodies not held in context. What changes is the **tool** (full-file replace, not targeted-edit) and the **payload size** (the whole file rather than one section's diff).

**Tradeoffs:**
- Per-call output grows from ~2K tokens (first section) to ~30K tokens (last section). Cumulative ~5× the targeted-edit path. Still well within gateway limits — nowhere near the 1M-token failure mode Section 10 was designed to prevent.
- Full-file replace is **atomic**: a failed write leaves the prior file intact. This is *safer* than the destructive failure mode of the broken targeted-edit operation.
- No `old_string` validation. Mitigation: the agent reads SPEC_FILE immediately before composing the payload (step 1), so it rebuilds from disk truth, not stale working memory. Skipping step 1 is a violation — the read is what replaces validation.

**Hard rules under fallback (from §10.5, still in force):** one section per inference call; never `Bash` + `sed` / `awk` / `python3 <<EOF` / `cat >` to mutate SPEC_FILE (the full-file replace operation is allowed, shell-based mutations are not); never compose multiple sections in memory before the write.

**When the runtime issue is resolved:** delete this subsection and return to the targeted-edit path defined in 10.4 and 10.5. The runtime fix is a two-line change: (1) UTF-8 encoding in the edit tool's write path, (2) atomic-write semantics so failed edits leave the prior file intact.

---

#### 10.5.2 Safe-Write Protocol (mandatory defensive backup for SPEC files)

**Why this exists.** Even with the discipline in §10.5 and the fallback in §10.5.1, a single bad targeted-edit call has been observed to truncate a SPEC file to 0 bytes mid-run (authored work wiped). The Safe-Write Protocol adds a `.bak` snapshot and a destructive-edit refusal gate so a single bad call cannot destroy authored work.

**Applies to:** any skill writing to `RESEARCH-SPEC-*.md`, `PLAN-SPEC-*.md`, `IMPL-STATE-*.md`, `REVIEW-SPEC-*.md`, or any other SPEC-shaped artifact the skill declares.

**Rule 1 — Defensive backup before any large edit:**

Whenever you must edit a SPEC file larger than 20 KB AND the edit changes more than 30% of the file's lines:

1. Compute: `lines_to_change` (new content vs current).
2. If `lines_to_change > 0.30 * total_lines AND file_size_kb > 20`:
   - `cp {target_file} {target_file}.bak.{ISO_TIMESTAMP}` via a single Bash call.
   - Proceed with the edit.
   - On success (post-edit file is non-empty AND > 50% of pre-edit size): delete the `.bak` file.
   - On failure or suspected truncation (post-edit file is < 50% of pre-edit size): restore from `.bak`, then emit a structured event:
     ```json
     {"event":"safe_write_restore","file":"<path>","reason":"post_edit_too_small","pre_size":<N>,"post_size":<M>}
     ```

**Rule 2 — Never `replace_all=true` on SPEC files.** Reinforces §10.5 "Section population: use `Edit`." Use targeted, anchored Edits with non-empty `old_string`. If a full rewrite is genuinely needed, write to a new file first, then `mv` atomically.

**Rule 3 — Refuse destructive `replace` calls (empty / too-short `old_string`):**

Before any `replace` / `Edit` whose `old_string` is empty (`""`) or shorter than 10 characters: REFUSE the operation. Emit:

```json
{"event":"safe_write_refuse","reason":"old_string_too_short_or_empty","file":"<path>","old_string_length":<N>}
```

This is the direct guard for whole-file overwrites caused by under-specified anchors.

**Rule 4 — Stray `.bak` cleanup at FIRST ACTION:**

In each section-shape skill's FIRST ACTION step (the `_progress.json` write described in §2), also glob `{output_path}/*.bak.*` and delete any files older than 1 hour. This bounds disk overhead from crashes that leave a `.bak` on disk.

**Relationship to §10.5 and §10.5.1.**
- §10.5 prevents bulk-rewrite cascades from `Bash` / `sed` / `python3`.
- §10.5.1 covers the non-ASCII-truncation failure mode (full-file replace as a workaround).
- §10.5.2 (this section) is the last line of defense — even if the agent uses the correct tool with the correct intent, the `.bak` snapshot survives a runtime malfunction.

---

#### 10.5.3 Tool-Output Retention (context-budget hard rule)

**Why this exists.** Running a command is a one-time cost; the expense is that its raw output then sits in the conversation and is **re-sent on every subsequent model round-trip** in the turn (and across turns until compaction). In an observed code excursion, a single tool-heavy turn (build + `npm test` + 14 edits) reached **6.9M input tokens** — not from re-reading across turns, but because verbose build/test logs and re-read source files rode along on every one of ~70 round-trips inside that one turn. The fix is not to run or log less. It is to **always execute fully, then retain only what the next round-trip needs.**

**The rule is about retention, never about execution. Always run the full command.** What you keep resident is decided by the output's *role*:

| Regime | Examples | Keep resident | Rationale |
|--------|----------|---------------|-----------|
| **Success** | build OK, tests green, lint clean, deploy healthy, migration applied | the **verdict only** — counts, exit code, the one summary line | A passing log carries no actionable detail; the body is pure noise. Full output is never needed. |
| **Failure** | failing test, compile error, probe 5xx, healthcheck down | the **failing slice** — the error, the stack, the failing names | The signal is the failure (usually <5% of the log). The passing lines before it are noise. Trim to the failure, not the whole run. |
| **Output *is* the evidence** | stack trace under root-cause, grep map being built, scanner findings, coverage report, diff under review | full fidelity **for this turn only**, then distill → drop | Sampling here can lose correctness, so capture full once, write the conclusion to the note/artifact, then drop the raw bytes — do not let them ride into the next round-trip. |

**The mechanism that satisfies "I sometimes need the full log" without paying to re-send it — redirect to a file, summarize into context, read on demand:**

```
npm test > .runlogs/test.txt 2>&1; tail -40 .runlogs/test.txt      # context gets the tail + counts
grep -A30 "FAIL src/work-orders" .runlogs/test.txt                  # full detail ONLY when a failure needs it
```

You never lose the full log — you stop re-billing it. Prefer the tool's own quiet/summary mode at the source where one exists (`jest` summary reporter, `--silent`, `pytest -q`, `gradle --quiet`, `npm --silent`), and on a run you expect to pass do **not** enable the runner's verbose / per-case-expanding mode (`--verbose`, `-vv`, `--reporter=spec`, etc.) — that prints a line per assertion and is the exact opposite of retention. Reach for verbose only after a failure, scoped to the failing target.

**Hard rules:**

- Never paste a large raw log (build/test/scan/probe) into the transcript when its outcome is success — emit the verdict line instead.
- On failure, emit only the failing portion; redirect the full log to a file and `grep`/`tail` it on demand.
- After any verification run, **distill the result (pass/fail counts, the exit verdict) into the working-set note or the artifact, then drop the raw output from working memory** — the same discipline §10.4.1 / §10.6 apply to re-read source.
- Do not re-read a file you just edited in the same turn to "re-orient" — reason from the edit result. (Observed: one service file read 4× in a single turn.)

**Enumeration guardrail (do NOT trim these away).** When the output is an enumeration-complete deliverable — a security scan's findings, a coverage-gap list, a toolchain inventory — every item matters and must not be sampled. The rule still applies, but "full fidelity" lands in the **artifact/output file**, not the chat transcript: write all findings to the file, keep only the count + the file path resident. Full is retained where it is the deliverable; it is simply not re-sent on every round-trip.

**Applies to:** any skill that runs build/test/lint/deploy/probe/scan commands inside a multi-round-trip turn. Cited by `conducting-excursion`, `implementing-code`, `fixing-bugs`, and the quick-lane `implementing-quick-change` (the quick-fix playlist runs `fixing-bugs`, so both quick lanes are covered); other tool-running skills (`bootstrapping-runtime-environment`, `launching-app`, `launching-subscription`, `tearing-down-services`, `managing-releases`, `researching-bug-fixing`, `reviewing-code`, `api-smoke-probe`, `browser-smoke-probe`, and the security/enumeration skills with the guardrail above) adopt the §10.5.3 citation as they are next touched.

---

### 10.6 CONTINUE on Re-Entry

If a skill is invoked while its `_progress.json` shows `status: RUNNING` with `skeleton_written: true` and a partial `sections` map, this is a CONTINUE invocation.

**Resume protocol (in order — do not skip steps, do not preload):**

1. Read `_progress.json` ONLY. Do not read anything else first.
2. Identify the first `pending` section in template order — call it `N`.
3. Read the SPEC_FILE ONCE to confirm the skeleton + already-populated sections match `_progress.json`. This is the only full SPEC_FILE read of the resume phase.
4. Enter Phase B at section `N`. From this point §10.4 + §10.4.1 govern — load only the extractions section `N` needs, write it via `Edit` (or §10.5.1 full-file replace under non-ASCII fallback), update `_progress.json`, drop, advance.

**Forbidden on re-entry (the failure mode this clause exists to prevent):**

- Re-reading all `_extractions/*.md` files upfront "to refresh context." Each extraction is read lazily in Phase B, only for the section that consumes it. A run that opens every extraction before the first `Edit` is violating this clause.
- Re-running Step 10.2 (Source Extraction Pass). Extractions are on disk from the prior run; the §10.2 REPAIR-reuse rule (lines 619–622) applies.
- Composing an upfront `think` block that plans all remaining `pending` sections. See §10.4.1 rule 2 — one section's plan per inference call, drafted immediately before that section's `Edit`.
- Re-reading raw source files. Already a §10.5 hard rule; restated here because re-entry is the iteration where it is most commonly violated.

**Symptom of a misbehaving resume.** If after step 3 your next 3 tool calls are all `view` calls against `_extractions/*.md` files, you are in the failure mode. Stop the cascade: pick the first pending section, load only its extractions, write it.

This is distinct from REPAIR mode (Section 7). REPAIR corrects already-completed specs based on `failure_feedback`. CONTINUE finishes in-progress ones based on `_progress.json` state.

---

### 10.7 Composition with Section 2 (FIRST ACTION / LAST ACTION)

Section 10 extends Section 2; it does not replace it.

- **FIRST ACTION (Section 2):** write `_progress.json` with `status: RUNNING` before any other file write. Section-shape skills initialize the `sections` map with all entries set to `pending` and `skeleton_written: false`.
- **Skeleton write (Section 10 Phase A):** within 5 tool calls of FIRST ACTION, also write the SPEC_FILE skeleton and flip `skeleton_written: true`.
- **LAST ACTION (Section 2):** after all sections are `complete`, set `status: COMPLETED` and `completed_at`.

---

### 10.8 Skill-Level Enforcement

Like FIRST / LAST ACTION (Section 2), every skill applying this protocol MUST include inline callouts — agents do not reliably follow protocol-only references for the generation discipline. The SKILL.md provides skill-specific contract bits that Section 10 cannot:

| Contract bit | Owned by SKILL.md | Owned by Section 10 |
|---|---|---|
| Section list in template order | Yes | No |
| Cross-section dependency graph | Yes | No |
| Extraction schema (which fields) | Yes | No |
| Trigger thresholds (10.2 defaults) | Override if needed | Default values |
| Skeleton / Edit discipline | No | Yes |
| `_progress.json` shape | No | Yes |
| CONTINUE semantics | No | Yes |

Recommended SKILL.md structure for section-shape skills:

1. **Step N.5 — Source Extraction Pass:** "Apply Section 10.2 with the extraction schema below: …" then provide the skill's extraction schema.
2. **Step N+1 — Generate Spec:** "Apply Section 10 Phase A then Phase B. Section list in template order: … Cross-section dependency graph: …"

---

## 11. Harness Output Sidecar

**Purpose.** The Stepwise harness reads each step's declared outputs from a sidecar JSON file written at a path the harness controls. If this file is missing, the orchestrator records `outputs: {}` in `state.json`, the UI's artifact panel renders empty, and downstream steps cannot pick up output values via propagation. **Artifacts on disk are not enough** — the sidecar is the framework contract between the skill and the orchestrator.

**Applicability.** Any skill that may be invoked by Stepwise. Standalone invocations (no orchestrator) do not need the sidecar — see the trigger gate below.

**When this section applies.** The input prompt contains a `## Run metadata` block with an `output_file = '<absolute path>'` line. If that block is absent (e.g. standalone invocation outside Stepwise), skip the sidecar write entirely.

---

### 11.1 Mechanism

```
1. Re-read the prompt's `## Run metadata` block. Copy the path on the
   `output_file = '...'` line VERBATIM. Do NOT reconstruct it from runId,
   session name, project name, or any other source — the path the harness
   wrote there is the path the harness expects to read from.

2. Re-read the prompt's `## Output parameters` table. For EVERY row in that
   table, include a key in the JSON object:
     - key   = the parameter name from the table
     - value = the resolved value the skill produced for that output
               (typically the path to the artifact folder or file)

3. Write that JSON object to the `output_file` path. This is the FINAL file
   write of the run — strict ordering (see Section 11.4):
     - AFTER any skill-defined output verification (e.g. file-existence checks).
     - AFTER Memory Bank session-end writes (Section 4).
     - AFTER `_progress.json` flipped to `status: COMPLETED` (Section 2 LAST ACTION).

4. Do NOT emit any tool call after the sidecar write. The next event in the
   transcript should be the `final_response` summary.
```

---

### 11.1.1 Path Correctness Rule (mandatory)

The value reported for any output **path** parameter MUST be the **actual on-disk path the skill wrote
to** — never a re-derived input parameter, never a fresh `value_template` substitution, and never a
scope-/sibling-adjusted path that was NOT the one actually used for the write.

```
BEFORE writing the sidecar, for every path-typed output parameter:
  1. Confirm the artifact exists at the path you are about to report
     (e.g. `ls <PATH>/<EXPECTED-FILE>*` or a stat/existence check).
  2. If the path the skill actually wrote to differs from the input parameter
     for ANY reason — sibling-folder rule, scope adjustment, feature_id stripping,
     SESSION_ID derivation — report the ACTUAL written path, NOT the parameter.
  3. Reporting a parameter you did not write to is the documented cause of the
     duplicate-artifact-on-REPAIR defect: a later REPAIR run is handed the wrong
     folder, finds no prior artifact, and silently rebuilds a duplicate. (See §7.)
```

**Worked example (sibling rule).** A planning step receives `planning_output_path =
/base/code-development/code-task-planning` but, because `feature_id` was empty and the upstream research
was scoped, Step 1 actually wrote to `/base/code-development/TICKET-001/code-task-planning/`. The sidecar
MUST report the latter (the path that holds the files), never the flat input parameter.

> **Note for skill authors:** "report the parameter VERBATIM" guidance (used by some skills to avoid
> pre-resolution double-prefixing) means *do not re-prepend project/scope segments yourself* — it does
> NOT license reporting a path you did not write to. When your own logic diverges from the parameter,
> the actual written path always wins.

---

### 11.2 Example

For a skill that declares a single output parameter named `foo_output_path`:

```json
{ "foo_output_path": "/abs/path/to/artifacts/outputs/<capability>/<step>" }
```

For a skill with multiple output parameters, include every row from the `## Output parameters` table — one key per row. The skill-specific parameter names live in each SKILL.md (Section 11.5); this section does not enumerate them.

---

### 11.3 Violation

Ending a Stepwise run with a `final_response` summary that lists the generated artifacts but no sidecar write breaks output propagation. The harness MAY have a `value_template` fallback for specific outputs, but that fallback is not guaranteed for every skill / every output. Treat the sidecar write as **non-optional** whenever the `## Run metadata` block is present in the prompt — do not rely on the fallback.

Skills whose outputs lack a `value_template` fallback fail **silently** when the sidecar is missing: downstream steps see `outputs: {}` and propagate empty values; no error surfaces in the run. This is the failure mode this section prevents.

---

### 11.4 Composition with Section 2 LAST ACTION

The sidecar write extends Section 2's LAST ACTION; it does NOT replace it. Strict ordering for any Stepwise-invoked skill:

1. All skill-specific outputs written and verified.
2. Memory Bank writes (Section 4 — `active-context.md`, `progress.md`).
3. `_progress.json` flipped to `status: COMPLETED` (Section 2 LAST ACTION).
4. **Sidecar write** (this section) — final file write of the run.
5. `final_response` summary — no further tool calls.

If the skill is invoked standalone (no `## Run metadata` block), step 4 is skipped. Steps 1–3 and 5 still apply.

---

### 11.5 Skill-Level Enforcement

Section 11 cannot enumerate output parameter names — those are declared per-skill in the capability YAML's `outputs` block and surface in each prompt's `## Output parameters` table. Each SKILL.md MUST include a short inline callout naming its output parameters, so the agent knows which keys to put in the sidecar JSON without parsing the YAML.

Recommended SKILL.md callout shape:

> **Step N: Emit harness outputs sidecar — apply execution-protocol.md Section 11.**
>
> **Output parameters this skill produces** (one key per row of the prompt's `## Output parameters` table):
>
> - `<param_name_1>`: <one-line description of the resolved value, e.g. "the resolved output folder path written in earlier steps">
> - `<param_name_2>`: ...
>
> Example sidecar contents (illustrative):
>
> ```json
> { "<param_name_1>": "/abs/path/...", "<param_name_2>": "..." }
> ```

The callout is intentionally short. Trigger gate, mechanism, ordering, and violation rule all live in Section 11 — the SKILL.md only names the keys.

---

## 12. Delegated Exploration

**Purpose.** Broad, read-only exploration — locating which files implement a feature, mapping every call site of a symbol, diffing two branches, enumerating what a PR changed, surveying naming conventions across a module, or sweeping a large set of input artifacts (briefs, PRDs, epics, prior specs) — can be **delegated to a read-only exploration subagent** instead of being done inline by the main agent. The subagent reads broadly and returns a compact conclusion; the main agent keeps its context lean for the synthesis and writing it alone is responsible for. This is an **optimization of the input-loading / research phase**, not a new deliverable.

**Why it helps.** It directly serves the same goal as §5 (FIC context monitoring): a fan-out read that would otherwise fill the main context with file contents instead returns as a short summary, so the main agent stays in its healthy utilization band and can do more synthesis before any compaction. Running the sweep on a cheaper/faster model (see §12.3) also makes exploration materially cheaper than reading every file in the main session.

**Applicability.** Any skill whose route includes §12. This section is **capability-gated and harness-agnostic**: it applies *only if the running harness exposes a read-only exploration subagent* (the mechanism and its name differ across harnesses — a built-in "explore"/"search" agent, a task/agent spawn tool, etc.). If the harness exposes no such capability, follow §12.5 (inline fallback) and change nothing else. **Never name a specific tool, agent, or model ID in a skill** — describe the capability and let each harness bind it.

**When this section applies in the run order.** Exploration happens during input loading / research, *before* spec generation (§10) or review. In the routing rule §12 therefore sits early — after §2, before §10 / §10.5.

---

### 12.1 When to delegate (and when not)

**Delegate when** the work is a broad, read-only sweep where you need the *conclusion*, not the raw contents:
- "Which files implement / touch capability X?"
- "Find every call site / consumer of symbol Y."
- "What changed in branch A compared to branch B / on this PR?"
- "What are the naming and structural conventions across this module?"
- "Across these N input artifacts, which ones mention requirement Z, and where?"
- Several independent search angles that can run in parallel.

**Do NOT delegate:**
- The **analysis, root-cause reasoning, decision, or spec-writing** — those are the skill's deliverable and stay with the main agent.
- Work that **mutates files** — the exploration subagent is read-only. Edits, scaffolding, and writes are never delegated through §12.
- Cases where **scope is already a single known file or a tight, named path** — just read it inline; a subagent round-trip adds latency for no context savings.
- Anything that would have the subagent make a **judgment the skill is accountable for** (e.g. "decide the fix", "rank the options"). Ask it for facts and locations, not verdicts.

The **execution-path / scope constraint each skill already enforces still applies** — delegating does not grant the subagent unrestricted repo access. Pass it the same bounded scope the main agent would have used.

---

### 12.2 The return contract

Instruct the subagent to return **conclusions plus `file:line` pointers and a compact summary — never file dumps or pasted source**. A good delegated prompt:
- States one bounded, specific question (not "explore the codebase").
- Names the scope / path constraint to stay within.
- Asks explicitly for: the answer, the precise `file:line` references that support it, and a short summary — and asks it NOT to paste large file contents back.

The returned summary is consumed as **research input** to the main session — the same role inline exploration notes would have played.

---

### 12.3 Model selection

Exploration is search and retrieval, not synthesis — a **cheaper / faster model tier is sufficient and materially cheaper**. When the harness lets you pick the subagent's model, prefer its fast/economy tier (the harness's equivalent of a "fast" or "mini" model) for §12 sweeps. Reserve the strong model for the synthesis the main agent does. Describe this as a preference for the cheap tier — do not hardcode a model ID, since the available tiers differ per harness.

---

### 12.4 Verify before use

Subagent output is **input, not ground truth**. The **Zero-Invention Policy still governs the main agent's deliverable**: before citing a `file:line` in a spec, basing a fix or plan step on a delegated finding, or asserting a fact the subagent reported, the main agent MUST confirm the pointer actually exists (open it / re-grep it). A subagent hallucination that flows unverified into the deliverable is still an invention by the skill. Treat delegated findings exactly as you would treat any unverified source: confirm, then use.

---

### 12.5 Fallback — no subagent capability

If the harness exposes no read-only exploration subagent, the skill performs the same exploration **inline**, bounded by its execution-path / scope constraint, exactly as before this section existed. §12 is **never a requirement and never changes the deliverable**: a skill must produce identical-quality output whether or not delegation was available. Do not block, warn, or degrade when the capability is absent — simply explore inline.

---

### 12.6 Skill-Level Enforcement

Each SKILL.md routed to §12 includes a short callout at its input-loading / exploration step pointing here. Recommended callout shape:

> **During input loading / exploration — apply execution-protocol.md Section 12 (Delegated Exploration) if your harness supports it.**
>
> Broad read-only sweeps for this skill (e.g. <one or two skill-specific examples: "locating the files on the bug's execution path", "diffing the feature branch against the base", "surveying which input artifacts cover each requirement">) MAY be delegated to a read-only exploration subagent on a cheap/fast model, which returns conclusions + `file:line` pointers (not file dumps). Synthesis, decisions, and all writing stay with this agent, which verifies any delegated `file:line` before using it (Zero-Invention still applies). If no subagent capability exists, explore inline under the usual scope constraint — output quality is identical either way.

The callout is intentionally short. The when/when-not rules, return contract, model preference, verification duty, and fallback all live in Section 12 — the SKILL.md only names the skill-specific sweep examples.

---

## 13. Code-Location Discipline

The context pack ships a `codebase-map.md` (directory layout, bounded-context ownership, key entry points, file responsibilities) **precisely so agents do not rediscover the repository structure with a full-tree scan on every run.** Observed failure: across a full code-development session — research, planning, implementation, code-review, and two excursions — `codebase-map.md` was **never opened**; it sat in the context-pack index as a one-line summary while agents located code by scanning the tree directly. This section makes the map a pre-scan step for code-searching skills.

### 13.1 The rule — consult before scan, not never scan

**Before running a repository-wide *discovery* scan to find where code lives** — a `grep`/`glob`/`find`/`ls -R` or directory walk whose purpose is "where is X?", "which files implement Y?", "what's the layout of Z?" — **first read `context-pack/codebase-map.md` (and `project-inventory.md` if present) and navigate from it.**

This is **"consult before scan," not "never scan."** The distinction is exact:

- **Forbidden first move:** an *exploratory* tree scan to discover layout/ownership/where-things-are, when the map already answers it. That is the scan the map exists to replace.
- **Always allowed:** **targeted reads** of the specific files the map points you to (open them, read them fully — that is the whole point of locating them). And a **fallback scan** when the map is **absent, or genuinely lacks the answer** (a file/area the map doesn't cover, or a question below the map's granularity).
- The map **replaces discovery scans, not targeted reads.** Finding the file via the map and then reading it is correct; grepping the whole tree to find the file the map already names is the waste.

### 13.2 When the map is insufficient — flag the gap

If `codebase-map.md` is missing the area you need, or is stale/contradicted by what you find, you **may** fall back to a scoped scan — and you **must record the gap** so the map can be corrected: name the path/area the map failed to cover in your output (the skill's open-questions / notes / findings, as appropriate). Silent fallback lets the map rot; a flagged gap is how it stays the cheap orientation surface. Never trust a stale map over ground truth — when the map and the code disagree, the code wins, and the disagreement is a flag.

### 13.3 Applies to (code-searching skills only)

This is **not** a global pre-flight read — it is targeted at skills that locate code. It does **not** apply to ideation, refinement, or other skills that never search a source tree.

Cited by: `researching-code-design`, `researching-feature-impl`, `researching-refactoring`, `researching-bug-fixing`, `planning-code-tasks`, `implementing-code`, `fixing-bugs`, `reviewing-code`, `conducting-excursion`, and the quick-lane skills `defining-quick-story` and `implementing-quick-change`. Each names §13 at its orientation / input-loading / localize step (the highest-value skills also make it an explicit Step-0 gate before any discovery scan). Skills not in this list do not read `codebase-map.md` on its account.

### 13.4 Composition with §12

§13 and §12 (Delegated Exploration) are complementary and ordered: **consult the map first (§13), then — only for what the map does not resolve — delegate the residual scoped sweep (§12).** The map shrinks what must be explored at all; delegation handles whatever exploration remains. A skill routed to both reads the map before it decides what (if anything) to fan out.
