# Labour Manifest — `forge`

> **Presence is permission.** Each entry below is one callable Labour
> unit that this repository exposes to a Forgejo Actions runner. An
> entry that is not in this file is not callable. There is no other
> registry, no other allow‑list, no implicit catch‑all.

This manifest is the boundary described in [`README.md`](README.md).
It is read by the Labour workflow (added by the operator who attaches
a runner — see [`README.md`](README.md) §1 and §5) and by any external
caller that wants to discover what this repo will do.

---

## How to read an entry

Every entry uses the same fields, in the same order:

| Field | Meaning |
| --- | --- |
| `id` | Stable identifier. Dot‑separated, lowercase, hyphenated. Prefix `labour.forge.` namespaces this repo's units. Never reused, never renamed. |
| `scope` | One of `private`, `local`, `society`. `private` units are unreachable from any surface; `local` units are callable by named external intelligences and human collaborators; `society` units are callable by any member of a Society this repo's operator has joined. |
| `entrypoint` | The `forge` subcommand or library symbol that implements the unit. Subcommands are reproduced verbatim as they appear in [`../README.md`](../README.md). |
| `inputs` | Typed object. Each field has a name and a JSON‑schema type. Inputs are validated before dispatch; mismatches `reject` the call. |
| `outputs` | Typed object. The runner workflow is responsible for shaping `forge`'s output into this shape before writing the result envelope. |
| `runner` | Required runner label. Calls that request a different label are not dispatched here. |
| `authority required` | Minimum authority the caller must assert: `read` < `draft` < `propose` < `act` < `govern` < `human`. |
| `review policy` | `auto` (run on dispatch), `intelligence-review` (an Intelligence in a Society must countersign), or `human-review` (a human collaborator must comment to release the call). |
| `lifecycle` | `oneshot` (default, no state between calls) or `kline` (long‑lived worker process keyed by a `kline.*` id). |
| `notes` | Optional. Free text. Not parsed. |

Authority mapping for this repo:

- `read` — call may only inspect remote forge state. No writes.
- `draft` — call may produce artefacts (e.g. a generated comment body)
  that the caller will choose whether to post.
- `propose` — call may create draft PRs, draft releases, or other
  reversible proposals on the *caller's* target forge.
- `act` — call may post comments, change labels, request reviews,
  approve PRs, edit notifications.
- `govern` — call may close or merge PRs, delete branches, edit
  protected refs.
- `human` — call must be initiated by a human collaborator; no
  Intelligence may assert this authority on its own.

---

## Units

### `labour.forge.repo-view`

- **scope:** `local`
- **entrypoint:** `forge repo view`
- **inputs:**
  - `host: string` — forge host (e.g. `github.com`). Optional; if
    omitted, the runner's `.forge` and `FORGE_HOST` resolution rules
    apply.
  - `owner: string`
  - `repo: string`
- **outputs:**
  - `repository: object` — the JSON form of the repository record.
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`
- **notes:** Pure read. Useful as a smoke test of the runner.

### `labour.forge.issue-list`

- **scope:** `local`
- **entrypoint:** `forge issue list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `state: enum("open", "closed", "all")` — default `open`.
  - `labels: array<string>` (optional)
- **outputs:**
  - `issues: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.pr-list`

- **scope:** `local`
- **entrypoint:** `forge pr list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `state: enum("open", "closed", "merged", "all")` — default `open`.
- **outputs:**
  - `pull_requests: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.pr-create`

- **scope:** `private`
- **entrypoint:** `forge pr create`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `title: string`
  - `head: string`
  - `base: string` (optional; defaults to the target repo's default branch)
  - `body: string` (optional)
  - `draft: boolean` — default `true` for this manifest entry.
- **outputs:**
  - `pull_request: object`
- **runner:** `forge-cli`
- **authority required:** `propose`
- **review policy:** `intelligence-review`
- **lifecycle:** `oneshot`
- **notes:** Held at `scope: private` until a concrete operator
  promotes it. PR creation is a *write* against an external forge and
  must not be silently callable from an unspecified caller.

### `labour.forge.pr-review-approve`

- **scope:** `private`
- **entrypoint:** `forge pr review approve`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `number: integer`
  - `body: string` (optional)
- **outputs:**
  - `review: object`
- **runner:** `forge-cli`
- **authority required:** `act`
- **review policy:** `human-review`
- **lifecycle:** `oneshot`
- **notes:** Approving a pull request on an external forge is a
  high‑authority act. This entry is shipped `private` and gated by
  `human-review` so that promotion is a deliberate, signed commit.

### `labour.forge.pr-reviewer-request`

- **scope:** `private`
- **entrypoint:** `forge pr reviewer request`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `number: integer`
  - `reviewers: array<string>`
- **outputs:**
  - `pull_request: object`
- **runner:** `forge-cli`
- **authority required:** `act`
- **review policy:** `intelligence-review`
- **lifecycle:** `oneshot`

### `labour.forge.release-list`

- **scope:** `local`
- **entrypoint:** `forge release list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
- **outputs:**
  - `releases: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.ci-list`

- **scope:** `local`
- **entrypoint:** `forge ci list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `branch: string` (optional)
- **outputs:**
  - `runs: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.ci-log`

- **scope:** `local`
- **entrypoint:** `forge ci log`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
  - `run_id: integer`
- **outputs:**
  - `log: string`
  - `truncated: boolean`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.branch-list`

- **scope:** `local`
- **entrypoint:** `forge branch list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
- **outputs:**
  - `branches: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.label-list`

- **scope:** `local`
- **entrypoint:** `forge label list`
- **inputs:**
  - `host: string` (optional)
  - `owner: string`
  - `repo: string`
- **outputs:**
  - `labels: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`

### `labour.forge.notification-list`

- **scope:** `private`
- **entrypoint:** `forge notification list`
- **inputs:**
  - `host: string` (optional)
  - `unread: boolean` (optional)
- **outputs:**
  - `notifications: array<object>`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `auto`
- **lifecycle:** `oneshot`
- **notes:** Notifications are scoped to the token holder. Held at
  `scope: private` because exposing them to anyone other than the
  token's owner would leak that owner's forge inbox.

### `labour.forge.notification-read`

- **scope:** `private`
- **entrypoint:** `forge notification read`
- **inputs:**
  - `host: string` (optional)
  - `id: string`
- **outputs:**
  - `notification: object`
- **runner:** `forge-cli`
- **authority required:** `act`
- **review policy:** `human-review`
- **lifecycle:** `oneshot`

### `labour.forge.api`

- **scope:** `private`
- **entrypoint:** `forge api`
- **inputs:**
  - `host: string` (optional)
  - `method: enum("GET", "POST", "PUT", "PATCH", "DELETE")` — default `GET`.
  - `path: string`
  - `body: object` (optional)
- **outputs:**
  - `status: integer`
  - `body: object`
- **runner:** `forge-cli`
- **authority required:** `govern`
- **review policy:** `human-review`
- **lifecycle:** `oneshot`
- **notes:** The `forge api` passthrough can reach any endpoint the
  token permits. It is shipped `private` for the same reason an
  unscoped shell is shipped `private`: there is no static authority
  level that bounds it. Promote per‑caller, with a typed wrapper
  unit, never as a general capability.

### `labour.forge.session` *(kline)*

- **scope:** `private`
- **entrypoint:** `internal/forges.NewClient` (held inside a worker
  process; addressed via a stable `kline.forge.session.<n>` id)
- **inputs (materialisation):**
  - `host: string`
  - `forge_type: enum("github", "gitlab", "gitea", "bitbucket")`
  - `base_url: string` (optional; required for self‑hosted)
- **outputs (materialisation):**
  - `kline_id: string`
- **runner:** `forge-cli`
- **authority required:** `read`
- **review policy:** `intelligence-review`
- **lifecycle:** `kline`
- **notes:** A warm forge client. Subsequent calls address the same
  `kline.forge.session.<n>` id and reuse its HTTP client, token
  cache, and rate‑limit window. Termination is an explicit call with
  `result.outputs.kline_closed: true` or removal of this manifest
  entry. Held `private` until the operator declares which callers may
  hold a session.

---

## Promotion and demotion

Changing `scope` is a normal commit to this file. Each transition is
reviewable, signable, and revertible:

- `private → local` — unit becomes callable from any external surface
  the operator has wired up (Surface A or B in
  [`External-Execution-Interfaces.md`](https://github.com/japer-technology/forgejo-society/blob/main/FORGEJO-SOCIETY-INTRODUCTION/analysis/External-Execution-Interfaces.md)).
- `local → society` — unit becomes callable across a Society the
  operator has joined (Surface C). **Do not promote any unit to
  `society` unless this repository's operator has explicitly chosen a
  Society membership.** A pure Labour repo with no Society membership
  has no Surface C and therefore no use for `society` scope.
- Any → `private` — unit becomes immediately uncallable from every
  surface.

Promotion does not change the entrypoint, runner, authority, or
review policy. Those must be edited deliberately.
