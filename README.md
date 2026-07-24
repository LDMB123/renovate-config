# LDMB123/renovate-config

Shared Renovate preset for the LDMB123 fleet — the sync plane of the
two-plane federation described in
`~/Developer/GitHub/_manifests/dependency-modernization-2026-07-22/PLAN.md`.
The policy plane (what versions/config *should* be) lives in `lj-shared`'s
published `@LDMB123/*` packages and reusable CI workflows; this repo is the
brain that keeps every repo's dependencies moving in lockstep with that
policy over time.

**Status:** created on GitHub (public, tagged `v1.0.0`) and onboarded to its
Phase 1 canary, `bor-radio`, whose `renovate.json` extends
`github>LDMB123/renovate-config#v1.0.0` alongside a still-live
`.github/dependabot.yml` — see `PLAN.md`'s Phase 1 gate ("Renovate opens the
expected grouped PR") for how this gets proven before fanning out further.

## Adopt (per repo)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>LDMB123/renovate-config"]
}
```

Retired tombstones (`coordinator-mcp-worker`, `openclaw-mcp-worker`,
`lj-shared-tail`) extend the disabled sub-preset instead:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>LDMB123/renovate-config:disabled"]
}
```

Per `PLAN.md` guardrail #4: keep `dependabot.yml` live in a repo until
Renovate has landed one real PR there — no double-PR window, and Dependabot
stays the fallback if Renovate misconfigures for that repo.

## What `default.json` does

- Extends `config:recommended` (Renovate's own baseline: autodetects npm,
  pip/pep621, cargo, github-actions, etc. by manifest presence — no manual
  per-ecosystem `enabled` toggles needed) plus the official
  `helpers:pinGitHubActionDigests` preset (SHA-digest pinning for GitHub
  Actions, per `PLAN.md` §Extended A).
- `rangeStrategy: pin` fleet-wide — matches the plan's bleeding-edge/lockstep
  stance; this is also why `lj-shared`'s new presets (`dev-preset` etc.) use
  caret ranges internally but the *fleet* still gets exact-pin PRs.
- `dependencyDashboard: true` — the single view for what's pending across the
  whole estate.
- Five `packageRules` groups: shared dev toolchain, Cloudflare Worker
  toolchain (wrangler + workers-types), MCP SDK, React, and wasm-bindgen (with
  a `prBodyNotes` rebuild warning for `emerson-violin-pwa`, per `PLAN.md`'s
  Rust/WASM verified-lane requirement).

## Known gap: `compatibility_date` has no Renovate datasource

The plan called for "regex managers for `compatibility_date` and
`.nvmrc`/`.python-version`." Verifying against current Renovate docs
(2026-07-23) found:

- **`.nvmrc`** and **`.python-version`** already have native Renovate
  managers (`nvm` and `pyenv` respectively), enabled by default. A custom
  regex manager for either would duplicate/conflict with the built-in one —
  correctly **not** added here.
- **`compatibility_date`** has no native manager (confirmed: no Cloudflare/
  wrangler entry in Renovate's manager list) *and* no clean custom-datasource
  mapping — Cloudflare doesn't publish compatibility dates as a queryable
  release feed, so a `customManagers` regex extraction would have nothing
  correct to compare `currentValue` against. A `github-tags` datasource
  against `cloudflare/workerd` was considered and rejected: workerd's tag
  format (`v1.YYYYMMDD.N`) doesn't compare cleanly against the bare
  `YYYY-MM-DD` value in `wrangler.jsonc`, so it would produce noise, not
  correct PRs.
- Separately, `compatibility_date` changes *runtime behavior* (it's an
  opt-in to new platform semantics, not a routine version bump) —
  `lj-shared/cf-worker-base/README.md` already documents a **manual**
  quarterly-sweep cadence for it. Auto-generating Renovate PRs here would
  silently override that existing doctrine without a decision to do so.

Left as an explicit, named gap rather than shipped half-verified. Revisit if
Cloudflare ever publishes a machine-readable compatibility-date feed, or if a
deliberate decision is made to automate the quarterly sweep via Renovate's
`dependencyDashboardApproval` (manual-trigger-only) gate instead of the
current fully-manual process.

## Verification

Proven at the `bor-radio` Phase-1 canary: onboard the repo, confirm Renovate
opens the expected grouped PR(s) with the right groupings and `rangeStrategy:
pin` behavior, confirm the Dependency Dashboard reflects the repo correctly,
then fan out per `PLAN.md`'s Phase 2/3 rollout.
