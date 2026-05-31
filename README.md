# trellis-workflows

Personal Trellis workflow marketplace for reusable `workflow.md` templates.

This repository publishes one stable workflow template:

- template id: `dzshzx-channel`
- base workflow: `channel-driven-subagent-dispatch`

The first release adds one narrow behavior change on top of the upstream
channel-driven workflow:

- A read-only architecture review/report pass may skip Trellis task creation
  when it does not modify business code, task state, or shared project config,
  and the only write is a local HTML report under
  `.scratch/reports/architecture-review-<timestamp>.html`.

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
    └── dzshzx-channel.md
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

Recommended update loop:

1. Fetch the latest upstream workflow template.
2. Diff it against `marketplace/workflows/dzshzx-channel.md`.
3. Merge structural and semantic upstream changes first.
4. Re-apply the narrow architecture-review exception if still needed.
5. Re-test in a throwaway repository with `trellis workflow --create-new`.
