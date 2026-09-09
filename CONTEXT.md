# GSA/api.data.gov context
> refreshed 2026-09-09 | upstream default: main @ 1908e4e (fork main identical)

## Identity & policies
- upstream: GSA/api.data.gov, default branch `main`, primary "language": Hugo (Go) site + Vue 3 metrics app (assets/javascripts/metrics). en-us.
- CLA/DCO: none. CONTRIBUTING.md = CC0 public domain waiver only. No contributor-signup requirement.
- AI-assisted PR policy: unstated (no AI_POLICY; no AI mention anywhere in CONTRIBUTING/README/.github; org GSA/.github default = 404).
- signed commits required: no.
- PR template: none (repo has no PULL_REQUEST_TEMPLATE; GSA/.github org default 404) -> use pipeline 4-section fallback body.
- external tracker: GitHub issues. Some issues live in 18F/GSA org trackers (e.g. ops) but GitHub is primary.

## Conventions (verified from merged PRs)
- branch naming: dominated by `dependabot/*`; human PRs use descriptive lowercase kebab, e.g. `updates-fixes`, `csp-update`, `signup-form-updates`, `updating-contact-page`. Prior fork PR used `fix/<kebab>`.
- commit style: imperative; recent human commits: "Update 18F references to GSA for GitHub org change.", "Fix one more 18F repo reference."
- test command: NONE (no test runner). CI (main.yml) only runs `pnpm lint --max-warnings 0` (eslint) and `pnpm run format:check` (prettier) inside a Hugo docker image. That is the substantive CI gate.
- how outside PRs get merged: maintainer Nick Muerdter (GUI) merges dependabot + small human edits; low volume, active (merged Jul 2026). Fork CI caveat: workflow `on: [push, pull_request]` does `docker buildx bake --push` to ghcr.io/gsa before lint/prettier; on a fork push permission fails so lint/prettier jobs may not run -> verify locally.

## Maintainer picture
- Active: Nick Muerdter (GUI), GSA. Recent human merges are small ref org (18F->GSA) + dependency updates.

## Issue-area health
- Open issues are mostly feature/backlog requests. #667 "API usage graphs always drop to zero" (open since 2023, no comments/labels/assignee = not triaged) documents the metrics chart behavior that led to the `data.hits.pop()` "hide current month" workaround in UsageChart.js. #703/#671 bug issues are assigned or in api-umbrella, not this repo.
- Route: no maintainer-engaged open issue survives -> repo-audit self-found gap.

## Gap ledger (dedupe — READ FIRST, never re-pick)
- 2026-08-24 self-found — dropped-no-tractable-issue (no unassigned verifiable bug then).
- 2026-08-26 PR #2 (fix/organizations-table-data) — pr-opened + pr-updated. OrganizationsTable `this.organizations` Vue3 setup bug. STILL OPEN on fork, NOT merged upstream. Do NOT re-pick.
- 2026-09-09 self-found gap — UsageChart.js mutates the `hits` prop (and the shared pinia store monthly array) via `data.hits.pop()` when title === "All Time". See Mined gaps. outcome: pr-opened (https://github.com/olitreadwell/api.data.gov/pull/3).

## Mined gaps (discovered, not yet attempted)
- 2026-09-09 docs/stale: residual `18F` references in docs/procedures.md point at the STILL-EXISTING 18F org repos/laptops/handbook (org move only relocated THIS repo to GSA; other repos are intact) -> NOT a bug, status: dropped(not-actual).
- 2026-09-09 clean-code/prop-mutation: `assets/javascripts/metrics/components/UsageChart.js` chartData computed does `data.hits = props.hits; if (title === "All Time") data.hits.pop();`. `pop()` directly mutates the prop array, which is a live reference into the pinia store's `hits.monthly` for the selected org ("all" included). Because the store getter returns the same array reference and recomputation happens on every org switch, each switch permanently pops an additional month from the shared store data, progressively eroding the historical "All Time" series within a session (issue #667 documents the underlying drop-to-zero that the pop is meant to mask). Fix: copy (`data.hits = props.hits.slice()`) before popping so only the displayed copy loses the current month. Repro: logic-level before/after (pop mutates the input array; slice+pop does not). status: attempted -> PR #3 opened 2026-09-09 (branch fix/metrics-chart-all-time-mutation, url https://github.com/olitreadwell/api.data.gov/pull/3). Verified: eslint --max-warnings 0 . clean, prettier --check clean, logic-level repro (store array 5->2 pre-fix, intact post-fix). Fork CI does not run workflow jobs (docker buildx --push targets GSA packages); substantive checks verified locally.
