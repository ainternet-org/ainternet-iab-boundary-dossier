# AInternet-in-a-box Boundary Dossier

Status: public-safe technical dossier draft.

This repository is the shareable boundary map for AInternet-in-a-box while the
private implementation repository is being tightened for release.

It is not a marketing site and not a complete source drop. It documents the
runtime boundary that matters for AI governance:

```text
reasoning proposes -> substrate folds facts -> admissibility verdict -> guarded execution -> receipt
```

The important claim is narrow:

> A model does not decide what it may do. The box decides whether a proposed
> action can carry under current identity, authority, posture, locality,
> machine facts, runtime binding and receipts.

## What This Dossier Contains

- `architecture/boundary-map.md` — where the substrate stops and policy layers begin.
- `architecture/layer-map.md` — L0/L1/L2/L3 placement.
- `operator-runbook.md` — terminal-first setup and actor examples.
- `test-vectors/valo-pass-degrade-halt.md` — VALO-style PASS/DEGRADE/HALT mapped to IAB verdicts.
- `evidence/` — redacted sample outputs for provision, launch and System-BOM.
- `pentest-notes/refusal-cases.md` — refusal behavior and dark reasons.
- `w3c-jis/standards-alignment.md` — W3C/JIS/VC alignment points.
- `glossary.md` — shared vocabulary.

## Repo Boundary

The private implementation repo is not the first artifact to send to reviewers.
This dossier is the first artifact: it lets a technical reviewer inspect the
contract, examples, and refusal semantics without receiving unfinished product
internals.

Implementation access can follow under an agreed scope.

## Quick Reading Path

1. Read `architecture/boundary-map.md`.
2. Read `test-vectors/valo-pass-degrade-halt.md`.
3. Inspect `evidence/provision-status-samples/local-only-codex-denied.json`.
4. Inspect `evidence/launch-verdict-samples/not-ready-egress-denied.json`.
5. Read `w3c-jis/standards-alignment.md`.

## Core Comparison

VALO public language frames a deterministic gate before execution, returning
PASS / DEGRADE / HALT.

IAB provides local substrate facts and native enforcement for the same shape:

| Gate language | IAB language | Meaning |
|---|---|---|
| PASS | `0x4000`, `can_carry=true` | action may proceed |
| DEGRADE | triage hold, partial posture, degraded sensor | action needs review or lower claim |
| HALT | `0x0000:<reason>` | action refused before execution |

The gate should not be fed by prompt promises. It should be fed by local facts.

