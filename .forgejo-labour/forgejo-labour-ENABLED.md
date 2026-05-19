# `forgejo-labour-ENABLED.md`

This file is the fail‑closed sentinel for the Labour layer of this
repository. Its meaning is defined entirely by its **presence**.

- **Present on a commit** → the Labour workflow on a connected
  Forgejo Actions runner is permitted to dispatch units declared in
  [`labour-manifest.md`](labour-manifest.md), subject to each unit's
  own `scope`, `authority required`, `review policy`, and `runner`
  fields.
- **Absent from a commit** → the Labour workflow refuses every call.
  No unit runs. No envelope is produced. No artefact is committed.
  The refusal is recorded as `result.status: rejected` with reason
  `sentinel missing` for any call that arrives on a sentinel‑less
  ref.

Deleting this file is the kill switch. It disables every Labour
surface immediately. It does not require an Intelligence, a Society,
a workflow change, a token rotation, or a runner restart.

This file holds no configuration. Its body is intentionally not
parsed. If you find yourself wanting to put settings here, put them in
[`labour-manifest.md`](labour-manifest.md) instead.

See [`README.md`](README.md) §5 for the maintainer's complete
withdrawal procedure.
