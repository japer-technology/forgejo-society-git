# Forgejo Labour — `forge`

An execution layer. It does not govern, reason, or hold intent.

<p align="center">
  <picture>
    <img src="https://raw.githubusercontent.com/japer-technology/forgejo-society/main/LOGO.png" alt="Forgejo Society" width="320">
  </picture>
</p>

It runs code, receives tasks, produces artefacts, commits results, and reports back.

---

## What this folder is

This is the **first pure `.forgejo-labour/` repo**.

> A pure Labour repo holds **only** the Labour doorway. It carries no
> `.forgejo-intelligence/` (no mind) and no `.forgejo-society/` (no
> federation). It is hands without a head: a public repository whose
> hand‑written code is declared callable by an outside Intelligence,
> through a Forgejo Actions runner that the maintainer controls.

The host repository (`japer-technology/forgejo-society-git`) is a Go
library and CLI — [`forge`](../README.md) — for working with git forges
(GitHub, GitLab, Gitea/Forgejo, Bitbucket Cloud) through a single
interface. That code is the *raw material*. This folder is the
**boundary** that decides which parts of it become callable Labour, on
what terms, on whose runner, and at what authority level.

Nothing in this folder reasons, plans, or talks to humans. Everything in
this folder is a typed contract that an external Intelligence — running
in a different repository, in a Society, or nowhere at all — can rely on
when it asks a Forgejo runner to *do* something with `forge`.

---

## The pure‑Labour stance

Per the canonical specification in
[`japer-technology/forgejo-society`](https://github.com/japer-technology/forgejo-society),
a repository may sit in one of four configurations:

| Configuration | Has Code | Has Intelligence | Has Labour | Has Society |
| --- | --- | --- | --- | --- |
| **Code only** | yes | no | no | no |
| **Code + Intelligence** | yes | yes | no | no |
| **Code + Intelligence + Labour** | yes | yes | yes | no |
| **Full Society member** | yes | yes | yes | yes |

This repository is deliberately **none of those**. It is a fifth, simpler
shape that the specification permits but does not yet exhibit:

> **Code + Labour, no Intelligence, no Society.**

It is *pure Labour*. There is nothing here to reason; there is nothing
here to federate. There is only `forge` and a typed declaration of how
`forge` may be invoked from outside.

Why this shape exists:

1. **`forge` is a tool, not a mind.** It is a thin, deterministic
   wrapper over forge APIs. It has no opinions, no goals, and no memory
   of its own. Attaching an Intelligence to it would be a category
   error: the cognition belongs to the caller, not to the wrench.
2. **Any public repo can be executed on Forgejo runners.** A long‑lived,
   auto‑restarting runner subscribed to this repository can hold a warm
   `forge` process (or one process per forge backend) and service calls
   indefinitely. That is the whole point of the Labour layer: turn an
   ordinary public repo into callable infrastructure for a greater AI
   system, without inviting that AI into the repo.
3. **Intent stays with the caller.** An Intelligence in another
   repository decides *why* a `forge` call is made. This repo only
   decides *whether* and *how* the call is allowed to run.

---

## How this repo is incorporated into a greater AI infrastructure

The greater AI infrastructure is assumed to be a Forgejo Society — or
any system that speaks the same call/result envelope — running on
hardware its operator controls.

### 1. The runner

A Forgejo Actions runner is registered against this repository (or
against the host organisation, scoped to this repository). The runner is:

- **Long‑lived.** It is a daemon, not an ephemeral container. It is
  expected to survive across many calls so that warm `forge` state
  (HTTP clients, token caches, rate‑limit windows) can be reused.
- **Auto‑restarting.** It is supervised (systemd, OpenRC, Docker
  restart policy, or equivalent). A crashed worker is replaced
  immediately; a maintenance restart is invisible to callers because
  pending calls are queued by Forgejo Actions, not by the worker.
- **Labelled.** It advertises a runner label (see
  [`labour-manifest.md`](labour-manifest.md), field `runner:`). Calls
  that request a label this runner does not carry are not dispatched
  to it.
- **Secret‑bearing, never secret‑leaking.** Forge tokens
  (`GITHUB_TOKEN`, `GITLAB_TOKEN`, `GITEA_TOKEN`, `BITBUCKET_TOKEN`,
  `FORGE_TOKEN`) live in Forgejo Actions secrets, are bound to the
  runner step only, and are never written into call or result
  envelopes. This is the same rule the Society uses elsewhere.

### 2. The boundary

The boundary is [`labour-manifest.md`](labour-manifest.md). Each entry
in it is one callable Labour unit, mapped to a `forge` subcommand or
library call. An entry declares:

- `id` — dot‑separated, lowercase, hyphenated (e.g.
  `labour.forge.repo-view`).
- `scope` — `private`, `local`, or `society`. Pure‑Labour repos
  realistically use `local` (callable by named external intelligences)
  or `society` (callable by any member of a Society the operator has
  joined). `private` units exist to be developed in place without ever
  being callable.
- `entrypoint` — the `forge` subcommand or Go function that implements
  the unit.
- `inputs` / `outputs` — typed shapes.
- `runner` — the required runner label.
- `authority required` — `read`, `draft`, `propose`, `act`, `govern`,
  or `human`.
- `review policy` — `auto`, `intelligence-review`, or `human-review`.
- `lifecycle` — `oneshot` or `kline` (long‑lived; see §4 below).

**Presence in the manifest is permission.** Removing an entry removes
the capability. There is no rollout, no UI, no race.

### 3. The call contract

Every external caller — whether a sister repo's Intelligence, a Society
event, or a human opening an issue — uses the same call envelope:

```
call.id          — call.{scope}.{labour-id}.{sequence}
call.target      — the labour.* id being invoked
call.caller      — agency.* / intelligence.* / human.* id
call.authority   — must be >= the unit's required authority
call.inputs      — typed inputs matching the manifest
call.context     — optional issue / PR / settlement pointers
call.constraints — runner label override, timeout, cost ceiling
```

…and receives the same result envelope:

```
result.id        — same shape as call.id
result.call      — originating call.id
result.status    — accepted | rejected | succeeded | failed | timeout
result.outputs   — typed outputs matching the manifest
result.artefacts — paths or content addresses of files produced
result.evidence  — log location, runner identity, commit SHA of the run
result.events    — event ids emitted during the run
```

Both envelopes are committed to whichever repository owns them, so the
audit trail is `git log`. The envelopes are transport‑independent: the
same shape is carried by in‑repo state files, by issue/PR slash
commands, and by federated Society events. See the Society's
[`External-Execution-Interfaces.md`](https://github.com/japer-technology/forgejo-society/blob/main/FORGEJO-SOCIETY-INTRODUCTION/analysis/External-Execution-Interfaces.md)
for the full spec.

### 4. Long‑lived units (klines)

Some `forge` work benefits from persistent state across calls — a
warm authenticated client, a paginated cursor, a rate‑limit budget
that survives between requests. Those units declare
`lifecycle: kline` in the manifest and address a stable `kline.*` id
across calls. The auto‑restarting runner is what makes this practical:
a kline is materialised inside a worker process that the runner
keeps alive; if the worker is restarted the next call rehydrates the
kline from its declared inputs.

For the purposes of this pure‑Labour repo, klines are the natural way
to expose, e.g., a long‑running `forge ci log --follow` stream or a
held forge session against a self‑hosted Gitea instance.

### 5. The kill switch

Execution is gated by the sentinel file
[`forgejo-labour-ENABLED.md`](forgejo-labour-ENABLED.md). The runner's
workflow refuses to dispatch any unit unless the sentinel is present
on the commit it is acting on. **Deleting the sentinel disables every
surface immediately**, with no exception for in‑flight calls beyond the
one currently executing on the runner.

A maintainer who wants to permanently withdraw this repository's
Labour from any greater AI infrastructure can:

1. Delete `forgejo-labour-ENABLED.md` — execution stops at once.
2. Demote every entry in `labour-manifest.md` to `scope: private` —
   even a re‑enabled sentinel would expose nothing.
3. Optionally delete this folder — the repository returns to being a
   plain public `forge` mirror with no callable surface at all.

---

## What is intentionally absent

- **No `.forgejo-intelligence/`.** This repo does not reason. If an
  Intelligence is needed to drive `forge`, it lives in the caller's
  repository.
- **No `.forgejo-society/`.** This repo is not a member of a
  federation. A Society may *call* it once Society‑scoped units are
  declared, but membership — settlements, governance, shared memory —
  is not installed here.
- **No `.forgejo/workflows/` additions in this PR.** The Labour
  workflow that actually executes manifest entries is added by the
  operator who attaches a runner; it is intentionally out of scope for
  the "first pure `.forgejo-labour/` repo" change, which only
  establishes the boundary. The Society's reference workflow
  (`forgejo-labour-AGENT.yaml`) is the recommended starting point.
- **No edits to existing `forge` code.** The boundary is additive.
  Nothing in `cmd/`, `internal/`, `gitea/`, `github/`, `gitlab/`, or
  `bitbucket/` changes. Removing this folder returns the repository
  exactly to its prior state.

---

## Files in this folder

| File | Purpose |
| --- | --- |
| [`README.md`](README.md) | This document. |
| [`labour-manifest.md`](labour-manifest.md) | Typed declarations of the callable Labour units this repo exposes. Presence is permission. |
| [`forgejo-labour-ENABLED.md`](forgejo-labour-ENABLED.md) | Fail‑closed sentinel. Presence enables execution; deletion disables every surface. |
| [`AGENTS.md`](AGENTS.md) | Vendor‑neutral guidance for any AI assistant editing files in this folder. |

---

## References

- [`japer-technology/forgejo-society`](https://github.com/japer-technology/forgejo-society)
  — the canonical specification, vocabulary, and reference Labour
  workflow.
- [`External-Execution-Interfaces.md`](https://github.com/japer-technology/forgejo-society/blob/main/FORGEJO-SOCIETY-INTRODUCTION/analysis/External-Execution-Interfaces.md)
  — the source of the manifest, envelope, and surface model used here.
- [`japer-technology/forgejo-society/.forgejo-labour/README.md`](https://github.com/japer-technology/forgejo-society/blob/main/.forgejo-labour/README.md)
  — the parent project's own Labour doorway, from which this folder
  inherits its identity.
- [`../README.md`](../README.md) — what `forge` actually is.

<p align="right">
  <picture>
    <img src="https://raw.githubusercontent.com/japer-technology/forgejo-society/main/LOGO.png" alt="Forgejo Society" width="80">
  </picture>
</p>
