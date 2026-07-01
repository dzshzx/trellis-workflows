# dzshzx Docs Ops Workflow

---

## Core Principles

1. **Docs are the deliverable** — this workflow targets pure-documentation repositories (runbooks / operational truth sources). "Implement" means editing owner documents; "check" means running the repo's validation entry and auditing convention compliance.
2. **Single source of truth** — one fact lives in exactly one owner document; everywhere else uses thin pointers. Convergence rules override convenience (see the repo's `conventions/convergence-policy.md`).
3. **Backlog and execution are split** — issue/PRD backlog lives in `.scratch/<feature-slug>/` (the repo's issue tracker); execution state lives in `.trellis/tasks/`. A backlog item graduates into a Trellis task when work starts; the task `prd.md` links back to its `.scratch` source.
4. **Main session edits by default** — documentation changes depend on global consistency (no duplication, thin pointers, cross-doc terms), which context-isolated sub-agents are bad at. Sub-agents are reserved for research fan-out and bulk mechanical edits.
5. **Live systems are read-only by default** — any mutation of the machine/remote state (install, restart, config write, key lifecycle) needs an explicit user-confirmed command list first, and a change-journal entry recording the process.
6. **Verify, then record** — the repo's validation script is the quality gate; time is written as absolute values; verification commands live next to facts.

## Repo Assumptions

This template assumes the target repository provides:

- `conventions/` — the rule set (convergence / classification / redaction / collection / verification / update policies). These are the real spec owner; `.trellis/spec/` only points at them.
- `scripts/check-runbooks.sh` (or equivalent) — the single executable validation entry.
- `.scratch/<feature-slug>/` — local markdown issue tracker with triage labels (see `docs/agents/issue-tracker.md`, `docs/agents/triage-labels.md`).
- `change-journal/` — non-authoritative process records for agent-assisted live changes (ADR 0009).
- `docs/adr/` — append-only architecture decision records.
- `inventory/` — current-state snapshot, source map, gaps, and stale-or-conflicting staging.

---

## Trellis System

### Developer Identity

Initialize your identity on first use:

```bash
python3 ./.trellis/scripts/init_developer.py <your-name>
```

### Spec System

`.trellis/spec/` holds thin pointers into the repo's real convention owners plus shared thinking guides:

```bash
python3 ./.trellis/scripts/get_context.py --mode packages
```

Long-term conventions belong in `conventions/` (the owner), never only in `.trellis/spec/` or agent files.

### Task System

Each task has its own directory under `.trellis/tasks/{MM-DD-name}/` with `task.json`, `prd.md`, optional `design.md`, optional `implement.md`, optional `research/`, and `implement.jsonl` / `check.jsonl`.

Common commands:

```bash
python3 ./.trellis/scripts/task.py create "<title>" [--slug <name>] [--parent <dir>]
python3 ./.trellis/scripts/task.py start <name>
python3 ./.trellis/scripts/task.py current --source
python3 ./.trellis/scripts/task.py finish
python3 ./.trellis/scripts/task.py archive <name>
python3 ./.trellis/scripts/task.py list [--mine] [--status <s>]
```

### Workspace System

Session journals live under `.trellis/workspace/<developer>/`:

```bash
python3 ./.trellis/scripts/add_session.py --title "Title" --commit "hash" --summary "Summary"
```

---

<!--
  WORKFLOW-STATE BREADCRUMB CONTRACT

  [workflow-state:STATUS] blocks are the single source for per-turn prompt injection.
  Do not delete tags or change tag syntax. The body can change; parsers should not.
-->

## Phase Index

```
Phase 1: Plan    -> classify the request, route backlog vs task vs direct edit
Phase 2: Execute -> edit owner docs (main session), validate continuously
Phase 3: Finish  -> check-runbooks gate, inventory/ADR sync, commit, wrap up
```

### Request Triage

Classify every turn into one of four lanes before doing anything:

1. **Question / lookup** — answer from the docs (INDEX-first navigation) or live read-only probes. No task, no edits.
2. **Routine update** — one owner document plus its `inventory/` / `INDEX.md` sync (the repo's `conventions/update-policy.md` Routine Update). No Trellis task: edit directly, run the validation script, commit.
3. **Agent-assisted live change** — the user delegates a machine/remote environment or config mutation. No Trellis task required for the change itself, but: mutations need an explicit user-confirmed command list first (collection policy), the process is recorded in `change-journal/` (ADR 0009), and resulting facts are written back to owner docs.
4. **Docs campaign** — multi-document convergence, repo-wide slimming, cross-module rule changes, or a new documentation area. This is Trellis-task territory: ask for task-creation consent and enter planning. If a `.scratch/<feature-slug>/` backlog item covers it, graduate that item instead of starting from scratch.

User approval to create a task is not approval to start implementation. Implementation waits until artifacts are reviewed and `task.py start` has run.

### Backlog Graduation (.scratch -> .trellis/tasks)

- `.scratch/<feature-slug>/` remains the backlog/issue owner: PRD drafts, triage labels (`needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, `wontfix`), and comment history.
- When a backlog item enters execution: create the Trellis task, write `prd.md` referencing the `.scratch` source path, and record the task path in the `.scratch` issue's `## Comments`.
- While the task is active, execution state (status, artifacts, research) lives only in `.trellis/tasks/` — do not mirror it back into `.scratch`.
- On archive, append the outcome (done / dropped, plus the commit hash) to the `.scratch` issue and update its `Status:` line.

### Planning Artifacts

- `prd.md` — requirements, constraints, acceptance criteria; links the `.scratch` source when one exists.
- `design.md` — for complex campaigns: target doc layout, owner/pointer boundaries, migration order, rollback shape.
- `implement.md` — ordered checklist, validation commands, review gates, rollback points.
- `implement.jsonl` / `check.jsonl` — context manifests. Point at `conventions/` files, `.trellis/spec/` indexes, and `research/` artifacts — not at the documents being edited.

Lightweight tasks may be PRD-only. Complex campaigns must have `prd.md`, `design.md`, and `implement.md` before `task.py start`.

[workflow-state:no_task]
No active task. Classify the turn into a lane first: (1) question/lookup — answer, no edits; (2) routine update (one owner doc + inventory/INDEX sync) — edit directly, run the validation script, no Trellis task; (3) agent-assisted live change — no Trellis task, but mutations need a user-confirmed command list first and a change-journal entry; (4) docs campaign (multi-doc convergence / rule change / new area) — ask for Trellis task-creation consent, prefer graduating an existing .scratch backlog item.
[/workflow-state:no_task]

### Phase 1: Plan

- 1.0 Create task `[required · once]` (docs-campaign lane only, after consent; link the `.scratch` source in `prd.md`)
- 1.1 Requirement exploration `[required · repeatable]`
- 1.2 Research `[optional · repeatable]`
- 1.3 Configure context `[conditional · once]` (sub-agent platforms only; point manifests at `conventions/` + research)
- 1.4 Activate task `[required · once]` (review gate, then `task.py start`)
- 1.5 Completion criteria

[workflow-state:planning]
Stay in planning; load `trellis-brainstorm` if requirements are unclear.
Docs campaigns: `prd.md` must state the owner/pointer boundary the change will leave behind (which doc owns which fact afterwards). Complex campaigns also need `design.md` (target layout + migration order) and `implement.md` (checklist + validation gates) before `task.py start`.
Read the repo's `conventions/` before designing: convergence-policy owns single-source/thin-pointer rules; update-policy owns the change flow.
[/workflow-state:planning]

[workflow-state:planning-inline]
Stay in planning; load `trellis-brainstorm` if requirements are unclear.
Docs campaigns: `prd.md` must state the owner/pointer boundary the change will leave behind. Complex campaigns also need `design.md` and `implement.md` before `task.py start`.
Inline mode: skip jsonl curation; Phase 2 reads `conventions/` and task artifacts directly.
[/workflow-state:planning-inline]

### Phase 2: Execute

- 2.1 Edit owner docs `[required · repeatable]` — main session by default
- 2.2 Validate `[required · repeatable]` — validation script + convention audit
- 2.3 Rollback `[on demand]`

[workflow-state:in_progress]
Main session edits owner docs directly (docs need global consistency; sub-agents are for research fan-out and bulk mechanical edits only — dispatch prompts start with `Active task: <task path>`).
Before editing: read the relevant `conventions/` files and the target doc's owner boundaries. Facts change in the owner doc first; other docs get thin pointers. Absolute dates only; secrets stay metadata-only.
After each edit batch: run the repo validation script (`scripts/check-runbooks.sh`). Live-system mutations inside a task still need a user-confirmed command list + change-journal entry.
Then: inventory/INDEX sync -> ADR if a rule-level decision was made -> commit (Phase 3.4).
[/workflow-state:in_progress]

[workflow-state:in_progress-inline]
Main session edits owner docs directly. Before editing read the relevant `conventions/` files; owner doc first, thin pointers elsewhere; absolute dates; secrets metadata-only.
After each edit batch run the repo validation script. Live-system mutations need a user-confirmed command list + change-journal entry.
Then: inventory/INDEX sync -> ADR if needed -> commit (Phase 3.4).
[/workflow-state:in_progress-inline]

### Phase 3: Finish

- 3.2 Debug retrospective `[on demand]`
- 3.3 Convention update `[required · once]` — new rules land in `conventions/` (owner), not `.trellis/spec/`
- 3.4 Commit changes `[required · once]`
- 3.5 Wrap-up reminder

[workflow-state:completed]
Changes committed. Archive the task (`task.py archive`), append the outcome to the `.scratch` source issue, and record the session.
[/workflow-state:completed]

### Rules

1. Identify the lane first; only the docs-campaign lane creates Trellis tasks.
2. Required steps inside a phase run in order and can't be skipped.
3. Phases can roll back (validation reveals a design defect -> return to planning, fix the artifact, re-enter execute).
4. The validation script must pass before any completion claim.
5. Conflicting facts found on the way go to the repo's stale-or-conflicting staging doc, not silently fixed in place.

### Active Task Routing

- Planning or unclear requirements -> `trellis-brainstorm`.
- Bulk mechanical edits or research fan-out -> dispatch `trellis-implement` / `trellis-research` with `Active task: <task path>` prefix.
- After edit batches -> run the validation script; `trellis-check` (agent or skill) for convention audits on large diffs.
- Repeated debugging -> `trellis-break-loop`; durable new conventions -> write to `conventions/` (owner), then `trellis-update-spec` only to refresh `.trellis/spec/` pointers.

### Guardrails

- Task creation approval is not implementation approval.
- Live systems default to read-only; every mutation class needs explicit user confirmation before execution.
- Secrets: record metadata only, never values.
- No relative-time wording in facts ("today", "recently"); use absolute dates with timezone.
- Commits happen only in this repository unless the user says otherwise; never push without an explicit ask.

---

## Phase 1: Plan

Goal: classify the request into a lane, and for docs campaigns produce reviewed planning artifacts before any editing.

#### 1.0 Create task `[required · once]`

Docs-campaign lane only, after task-creation consent:

```bash
python3 ./.trellis/scripts/task.py create "<task title>" --slug <name>
```

`--slug` is the human-readable name only; the `MM-DD-` prefix is added automatically. If the campaign comes from a `.scratch` backlog item, note the graduation in that issue's `## Comments` and link the issue path in `prd.md`.

Run only `create` here — `start` happens at step 1.4 after artifact review.

#### 1.1 Requirement exploration `[required · repeatable]`

Load `trellis-brainstorm` when requirements are unclear. For docs campaigns, the PRD must answer:

- Which facts move, and which document becomes their owner afterwards?
- Which documents become thin pointers, and what do they point at?
- What does "done" look like in terms the validation script + a reader can verify?

#### 1.2 Research `[optional · repeatable]`

Research fan-out may use sub-agents (`trellis-research`) or direct reads. Findings are persisted to `{TASK_DIR}/research/` — conversations get compacted, files don't. Typical research: current owner/pointer graph for the affected facts, live-probe outputs, prior ADRs touching the same rules.

#### 1.3 Configure context `[conditional · once]`

Sub-agent-dispatch platforms: curate `implement.jsonl` / `check.jsonl` with the relevant `conventions/` files, `.trellis/spec/` indexes, and `research/` artifacts. Inline platforms skip this; Phase 2 reads the same files directly.

#### 1.4 Activate task `[required · once]`

After artifact review:

```bash
python3 ./.trellis/scripts/task.py start <task-dir>
```

#### 1.5 Completion criteria

| Condition | Required |
|------|:---:|
| `prd.md` exists and states the post-change owner/pointer boundary | ✅ |
| User confirms the campaign should enter execution | ✅ |
| `task.py start` has run (status = in_progress) | ✅ |
| `design.md` + `implement.md` exist (complex campaigns) | ✅ |
| `research/` has artifacts (complex campaigns) | recommended |

---

## Phase 2: Execute

Goal: move the docs to the planned end-state while keeping every intermediate commit-able.

#### 2.1 Edit owner docs `[required · repeatable]`

Default: the main session edits directly. Read order before touching a file:

1. The relevant `conventions/` files (convergence, classification, redaction at minimum)
2. The affected area's `INDEX.md` and the target doc's current owner boundaries
3. Task artifacts (`prd.md` -> `design.md` -> `implement.md`) and `research/`

Editing rules: owner doc first, thin pointers after; required fields per the classification policy (`Sources:`, `Last verified:`, verification block); absolute dates; secrets metadata-only. Sub-agent dispatch is reserved for bulk mechanical edits — when used, the dispatch prompt starts with `Active task: <task path>` and states explicit write boundaries.

Live-system mutations that surface mid-task (e.g. verifying a fact requires fixing the system) do not inherit approval from the task: list the commands, get explicit user confirmation, record in `change-journal/`.

#### 2.2 Validate `[required · repeatable]`

```bash
scripts/check-runbooks.sh
```

Plus a convention audit on the diff: no duplicated facts, no dangling pointers, no relative dates, no secret values, inventory/INDEX consistent with the moved facts. The final pass before Phase 3.4 must run full-scope on everything the task touched, not just the latest batch.

#### 2.3 Rollback `[on demand]`

- Validation reveals a design defect -> return to Phase 1, fix the artifact, re-enter 2.1.
- An edit contradicts a live probe -> the live output wins; stage the conflict in the repo's stale-or-conflicting doc and re-plan that fact's move.

---

## Phase 3: Finish

Goal: leave the repo consistent, decisions recorded, and the work committed.

#### 3.2 Debug retrospective `[on demand]`

If the task hit the same failure repeatedly, load `trellis-break-loop` and capture the root cause before wrapping up.

#### 3.3 Convention update `[required · once]`

If the task produced a durable rule (not just moved facts): write it into `conventions/` — the owner — and append an ADR when it changes cross-module behavior or reverses a prior decision. ADRs are append-only; never rewrite old ones. `.trellis/spec/` gets pointer refreshes only. Even if the conclusion is "nothing to update", walk through the judgment.

#### 3.4 Commit changes `[required · once]`

1. `git status --porcelain` — snapshot dirty paths; classify AI-edited vs unrecognized (never silently include the latter).
2. Learn message style from `git log --oneline -5`.
3. Draft a batched commit plan (logical units, work commits before bookkeeping commits), present once for one-shot confirmation.
4. On confirmation: `git add` + `git commit` per batch. No amend. Never push (this repo may have no remote; pushing anywhere needs an explicit user ask).

#### 3.5 Wrap-up reminder

Archive the task (`python3 ./.trellis/scripts/task.py archive <name>`), append the outcome + commit hash to the `.scratch` source issue, and record the session with `add_session.py`.

---

## Customizing

This file is the local workflow contract; scripts are parsers only. Tag blocks (`[workflow-state:STATUS]`) drive the per-turn breadcrumb — keep opening/closing STATUS strings identical. When this template changes upstream (`trellis-workflows` marketplace), re-run `trellis workflow` in each target repo to pick it up; local edits between refreshes are allowed but will be overwritten on the next pull.
