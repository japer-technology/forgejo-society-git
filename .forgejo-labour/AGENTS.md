# AGENTS.md — `.forgejo-labour/`

Vendor‑neutral guidance for any AI assistant editing files in this
folder. This file applies to **this folder only**. It does not bind
anything in the rest of the repository.

## Identity of this folder

This is the Labour doorway of a **pure Labour repo**. There is no
`.forgejo-intelligence/` and no `.forgejo-society/` in this
repository, and **none must be added in the same change** that edits
this folder. The "Code + Labour, no Intelligence, no Society" shape
is the whole point of this repository's existence as the first pure
`.forgejo-labour/` repo (see [`README.md`](README.md), "The
pure‑Labour stance").

If an agent is asked to "add reasoning here", "wire this folder into
a Society", or "let this folder make decisions", the correct response
is to refuse and direct the request to the caller's Intelligence
repository instead.

## What may be edited

- [`README.md`](README.md) — concept and operator guidance. Edits
  must preserve the pure‑Labour stance and the references to the
  parent project.
- [`labour-manifest.md`](labour-manifest.md) — the boundary. New
  units may be added, existing units may have their `scope`,
  `inputs`, `outputs`, `authority required`, `review policy`,
  `runner`, or `lifecycle` changed. **`id` and `entrypoint` are
  effectively immutable**: changing them is equivalent to removing
  the old unit and adding a new one, and must be done in two commits
  for auditability.
- [`AGENTS.md`](AGENTS.md) — this file. Update when the editing
  rules genuinely change, not to record session‑specific notes.

## What must not be added

- No workflow files. The Labour workflow that executes manifest
  entries belongs in `.forgejo/workflows/` and is the operator's
  responsibility; it is intentionally out of scope for this folder.
  The user's original instruction was *"files only in a new folder
  called .forgejo-labour"* and that constraint stands for every
  subsequent edit of this folder.
- No secrets. Forge tokens live in Forgejo Actions secrets and are
  bound to runner steps. They must never appear in the manifest, in
  call or result envelopes, or in any markdown in this folder.
- No state. Files under `.forgejo-labour/state/` (call envelopes,
  result envelopes, kline records) are written by the runner at
  execution time, not committed by hand and not committed by agents
  editing this folder.
- No reasoning. Plans, deliberations, reading paths, essays, and
  agent dialogue do not belong here. They belong in an
  `.forgejo-intelligence/` folder in whichever repository hosts the
  caller. If a unit needs explanation beyond the manifest's `notes`
  field, prefer a short paragraph in `README.md` over a new file.

## How to promote a unit

Promotion is a `scope:` edit to [`labour-manifest.md`](labour-manifest.md)
and nothing else.

1. Verify the unit's `authority required` matches the worst caller
   that the new scope admits. A unit promoted from `local` to
   `society` may be reached by every member of the Society; its
   authority requirement must reflect that.
2. Verify the unit's `review policy` is still appropriate. Promotion
   often justifies raising the policy from `auto` to
   `intelligence-review`, or from `intelligence-review` to
   `human-review`.
3. Make the edit in a single commit whose message names the unit and
   the transition (e.g. `labour.forge.repo-view: local → society`).
4. Do not change the `entrypoint`, `inputs`, `outputs`, or `runner`
   in the same commit.

Demotion follows the same rules in reverse, and may be done at any
time without coordination — demotion is fail‑closed by construction.

## How to add a unit

1. Pick an `id` under the `labour.forge.` namespace that does not
   already exist in [`labour-manifest.md`](labour-manifest.md).
2. Map it to a real `forge` subcommand or library symbol that exists
   in this repository at the commit being edited. Do not declare
   units for code that has not been written yet.
3. Choose the most restrictive defensible values for every other
   field. New units should default to `scope: private`,
   `review policy: human-review`, and the lowest `authority required`
   that lets them function. Promotion is cheap and reviewable;
   over‑exposing a new unit on its first commit is not.
4. Add a `notes:` paragraph if and only if the unit's semantics are
   non‑obvious from its name, entrypoint, and inputs.

## Out of scope

Anything not covered above is out of scope for this folder. In
particular:

- Changes to `cmd/`, `internal/`, `gitea/`, `github/`, `gitlab/`,
  `bitbucket/`, or any other Go source — out of scope.
- Changes to top‑level `README.md`, `LICENSE`, `go.mod`, `go.sum`,
  `.golangci.yml`, or `.goreleaser.yml` — out of scope.
- Changes to `.github/` or `.forgejo/` directories — out of scope.

An edit that needs to touch any of those paths is not an edit to the
Labour boundary; it is a different change and must be raised as a
separate pull request against the appropriate part of the codebase.
