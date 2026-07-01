# trellis-workflows

Personal Trellis workflow marketplace for reusable `workflow.md` templates.

This repository publishes two stable workflow templates:

- template id: `dzshzx-channel`
  - base workflow: `channel-driven-subagent-dispatch`
  - adds one narrow behavior change on top of the upstream channel-driven
    workflow: a read-only architecture review/report pass may skip Trellis
    task creation when it does not modify business code, task state, or
    shared project config, and the only write is a local HTML report under
    `.scratch/reports/architecture-review-<timestamp>.html`.
- template id: `dzshzx-docs-ops`
  - base workflow: `native` (restructured for pure-documentation repositories)
  - target: runbook / operational truth-source repos (e.g. `ops-runbooks`)
    with `conventions/` as the real spec owner, a `.scratch/` markdown issue
    tracker, `change-journal/` for agent-assisted live changes, and a single
    validation entry such as `scripts/check-runbooks.sh`.
  - key behavior changes: four-lane request triage (question / routine update
    / agent-assisted live change / docs campaign — only campaigns create
    Trellis tasks); `.scratch` backlog graduates into `.trellis/tasks/`
    execution tasks; main-session editing is the default (sub-agents reserved
    for research fan-out and bulk mechanical edits); the repo validation
    script is the quality gate; live-system mutations always require a
    user-confirmed command list plus a change-journal entry.

## Important Runtime Model

Trellis does not live-reference this repository at runtime.

When a target repository selects `dzshzx-channel`, Trellis fetches the template
and writes it into that target repository's local `.trellis/workflow.md`.

If this marketplace template changes later, existing repositories do not update
automatically. Re-run `trellis workflow` in each repository that should pick up
the new version.

## Repository Layout

```text
marketplace/
├── index.json
└── workflows/
    ├── dzshzx-channel.md
    └── dzshzx-docs-ops.md
```

In `marketplace/index.json`, workflow entry paths are relative to the
marketplace root passed to Trellis (`gh:dzshzx/trellis-workflows/marketplace`).
For this repository that means the template path is
`workflows/dzshzx-channel.md`.

## Usage

### New Repository

```bash
trellis init \
  --workflow dzshzx-channel \
  --workflow-source gh:dzshzx/trellis-workflows/marketplace
```

Add your normal platform flags such as `--codex` and identity flags such as
`-u <your-name>` as needed.

### Existing Repository

```bash
trellis workflow \
  -t dzshzx-channel \
  -m gh:dzshzx/trellis-workflows/marketplace
```

### Safe Preview

```bash
trellis workflow \
  -t dzshzx-channel \
  -m gh:dzshzx/trellis-workflows/marketplace \
  --create-new
```

This writes `.trellis/workflow.md.new` in the target repository and leaves the
active workflow unchanged until you review the diff.

## Maintenance

Keep `dzshzx-channel` as a minimal-diff fork of the upstream
`channel-driven-subagent-dispatch` workflow.

`dzshzx-docs-ops` is a restructured fork of the upstream `native` workflow:
its phase/tag contract (`[workflow-state:STATUS]` blocks, three phases,
task.py lifecycle) tracks upstream, but the step bodies are rewritten for
documentation repositories. When upstream changes the tag contract or
task lifecycle, port those structural changes; do not port code-oriented
step bodies.

Recommended update loop:

1. Fetch the latest upstream workflow template.
2. Diff it against `marketplace/workflows/dzshzx-channel.md`.
3. Merge structural and semantic upstream changes first.
4. Re-apply the narrow architecture-review exception if still needed.
5. Re-test in a throwaway repository with `trellis workflow --create-new`.
