# CI Surface — Decision of Record

> Which repo certifies work for the merchant-routing engine, which CI lanes
> exist, and why the certifying surface is a *public* repo. The lane landed
> 2026-09-25; this doc records the decision 2026-10-13 and closes fleet plan
> Phase 12.2 for this repo explicitly. Board-facing statement; the lane's
> operational detail lives in [`.ngit/README.md`](../.ngit/README.md).

## Decision

**Engine work is certified by this public repo —
`felixfelix-bot/merchant-routing-engine` — via its Nostr CI (ngit-ci) lane
`.ngit/act/workflows/python-test.yml`.** There is no other certifying
surface: this repo deliberately carries no `.github/workflows/`, and the
private fleet repo that historically held engine work cannot carry ngit CI
at all (see [Policy](#policy-why-a-public-repo-certifies-engine-work)).

## Lanes

| Surface | Engine | Status |
|---|---|---|
| `.ngit/act/workflows/python-test.yml` | ngit-ci coordinator (Nostr; trust fleet coordinator `765cd47b…` / DQ05) | **Active — the only lane.** pytest suites + syntax checks |
| `.github/workflows/` | GitHub Actions | None exist; deliberately not used |

What the lane runs (pytest suites + syntax checks — 1912 + 33 + 91 passing
at the lane commits, measured 24-module ignore list), how to trigger it, and
how to bind a kind-9842 result to a signer:
[`.ngit/README.md`](../.ngit/README.md). Gate-evidence line shape:
`ci_evidence repo=merchant-routing-engine head=<40-hex> ref=<ref>`.

## Policy: why a public repo certifies engine work

The fleet's dual-push rule (**D-124 / D-128**: *public → GitHub + ngit;
private → GitHub only*) exists because ngit has no private mode — anything
pushed to Nostr is public forever. Consequences, fleet policy since
2026-09-14 and unchanged:

1. **`felixfelix-bot/hermes-orchestration` (private) is OUT OF SCOPE for
   ngit mirroring and CI.** It is never mirrored, never CI-triggered, and
   cannot be the certifying surface for anything. That is not a gap to
   close later — it is the policy.
2. **Therefore this public repo is the certifying surface for engine
   work.** Work that must show `ci_evidence` publishes its sanitized delta
   here and lets the lane run at that commit.

## Scope of the certificate (honesty clause)

A green run certifies **this public tree at the exact commit of the run** —
not the live engine tree, which is ahead of this repo. Measured on the
authoring fleet node 2026-10-13:

| File | Live (private, `~/.hermes/bot/`) | This repo |
|---|---|---|
| `zai_proxy.py` | 10 197 lines | `production/zai_proxy.py`, 7 852 lines |
| `flat_router.py` | 3 513 lines | `flat_router.py`, 1 338 lines |
| `garbage_detector.py` | 653 lines | absent |

A card changing the live engine must publish the sanitized delta here (no
`.env`, no keys, no `nsec` — this repo is public and the policy is
fail-closed) and re-run the lane before its `ci_evidence` can be honest.
Full per-file notes and the excluded `tests/` modules:
[`.ngit/README.md` § Scope](../.ngit/README.md#scope-what-a-green-run-does-and-does-not-certify).

**Evidence freshness (2026-10-13):** `ngit_ci_evidence.py
merchant-routing-engine` currently returns **no** kind-9842 rows from
`wss://relay.ngit.dev` / `wss://gitnostr.com` (relay retention; the green
runs of 2026-09-25 at `2ccbbb91` and `4e55c8f`, event ids recorded in
`.ngit/README.md`, are no longer served). When gate evidence is needed,
re-trigger the lane at the commit being certified and quote the fresh
kind-9842 — do not quote this doc as a substitute for a run.

## Fleet plan Phase 12.2 — CLOSED for this repo

`hermes-orchestration/docs/PLAN-fleet-governance.md` Phase 12.2 ("Add ngit
CI or a documented local pre-push gate for the 19 CI-less repos …
`merchant-routing` …") is **explicitly CLOSED for merchant-routing-engine**
as of 2026-09-25 by the lane above: maintainer identity `npub1nng5mx…`
(hex `9cd14d9acdbab7ca162b772c91fa514da8e3248f61f924bf3441e6e72c5f0dee`),
green kind-9842 at the lane commit on the ngit mirror (`2ccbbb91` on
`refs/heads/main`; `4e55c8f` on `refs/heads/ci/ngit-workflow-lane`). The
remaining 18 CI-less repos in that item stay open in the fleet plan — this
closure covers this repo only.
