# leartech-renovate-config

Shared Renovate config presets for all leartech repos.

## Why

Before this repo, every leartech repo had its own ~50-line `renovate.json` with the same boilerplate (extends, schedule, group names, customManagers) plus a few per-repo overrides. Result: 15 copies of mostly-the-same config, drift over time, and missing settings (e.g. `pinDigests`) that we'd need to add 15 times to fix once.

This repo holds the canonical config. Consumer repos shrink to 3–5 lines — they `extends` from here and add only their own overrides.

## Usage

In a consumer repo's `renovate.json`:

### Pure base (gitops repos, dockerfile repos, the catalog itself)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>mikelear/leartech-renovate-config"]
}
```

### Go service

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>mikelear/leartech-renovate-config",
    "github>mikelear/leartech-renovate-config:go-service"
  ]
}
```

### Angular app

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "github>mikelear/leartech-renovate-config",
    "github>mikelear/leartech-renovate-config:angular-app"
  ]
}
```

## What's in default.json

- `config:recommended` upstream baseline
- Weekly schedule: Mondays before 9am
- `platformAutomerge: true` — uses GitHub's auto-merge so PRs land as soon as checks go green
- **`pinDigests: true`** — every Dockerfile `FROM tag` is auto-pinned to `FROM tag@sha256:...`, and Renovate opens a PR whenever the upstream digest changes (catches mutated tags like `mcr.microsoft.com/playwright:v1.59.1-jammy` getting republished)
- Groups for patch (auto-merge), minor (manual), helm chart updates, docker base image updates
- `customManagers` regex for `image:` refs in `.lighthouse/` pipeline yaml so consumers' catalog task pins (e.g. `ghcr.io/mikelear/playwright-runner:0.25.3`) get bump PRs

## What `go-service.json` adds

- Auto-merge group for `ghcr.io/mikelear/leartech-go-runtime` image bumps
- Auto-merge group for `leartech-go-common` + `leartech-go-packages/*` gomod updates with `at any time` schedule (overrides the weekly default — internal libs need to flow within hours)

## What `angular-app.json` adds

- Auto-merge group for `ghcr.io/mikelear/leartech-nginx` image bumps
- Auto-merge group for `@mikelear/leartech-*` npm packages with `at any time` schedule
- Manual-review group for `@angular/*` (they version in lockstep, build verification needed)
- Group for lint tooling (eslint family)
- Group for test tooling (karma + jasmine)

## How to test changes to this repo

Renovate fetches presets via `raw.githubusercontent.com` on every run, so a change to `default.json` here is picked up by every consumer's next Renovate run (within ~5 min after the consumer's webhook fires).

To force a consumer to re-run sooner:
1. Push any commit to the consumer's `renovate.json` (even a no-op formatting change) — that triggers an immediate Renovate webhook
2. Or: comment `@renovate-bot retry` on an open Renovate PR in the consumer
3. Or: tick the "Check this box to trigger a request for Renovate to run again" checkbox on the consumer's Dependency Dashboard issue

## Related

- `~/leartech/hub/shared-rules/build-speed.md` — three drift defenses captured from the playwright-runner debug session that motivated `pinDigests`
- `~/leartech/hub/shared-rules/registry-cache.md` — the alternative path mqube uses (per-cluster pull-through cache)
