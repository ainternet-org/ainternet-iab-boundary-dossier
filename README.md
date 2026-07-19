# AInternet-in-a-Box — Boundary Dossier

**A model does not decide what it may do. The box does — from local facts, before execution, with a receipt.**

Status: public-safe technical dossier. This is the shareable boundary map while the private
implementation repository is tightened for release. It is **not** a marketing site and **not** a full
source drop. It documents the one thing that matters for AI governance: the runtime boundary between
*reasoning* and *execution*.

```text
reasoning proposes  →  substrate folds facts  →  admissibility verdict  →  guarded execution  →  receipt
```

The claim is narrow and testable:

> A model does not decide what it may do. The box decides whether a proposed action can carry under
> current identity, authority, posture, locality, machine facts, runtime binding and receipts — and
> enforces that verdict **natively, before anything runs**.

## Don't just read prose — run it

The boundary is not a diagram; it is a running verdict. On a box:

```sh
./box provision status --json   # authority · posture · admissibility (carry_decision/blocked_by) · next command
./box launch --json             # the fail-closed launch verdict

# a loopback Ollama actor carries with no egress; a Codex CLI actor (traffic → api.openai.com)
# fails CLOSED under a local-only posture until posture is raised AND egress is granted:
./box bind codex                # → 0x0000:egress-not-permitted
```

Redacted samples of exactly these outputs live in [`evidence/`](evidence/).

## What this dossier contains

- [`architecture/boundary-map.md`](architecture/boundary-map.md) — where the substrate stops and policy layers begin.
- [`architecture/layer-map.md`](architecture/layer-map.md) — L0/L1/L2/L3 placement.
- [`operator-runbook.md`](operator-runbook.md) — terminal-first setup and actor examples.
- [`test-vectors/valo-pass-degrade-halt.md`](test-vectors/valo-pass-degrade-halt.md) — PASS/DEGRADE/HALT mapped to IAB verdicts.
- [`evidence/`](evidence/) — redacted sample outputs for provision, launch and System-BOM.
- [`pentest-notes/refusal-cases.md`](pentest-notes/refusal-cases.md) — refusal behavior and dark reasons.
- [`w3c-jis/standards-alignment.md`](w3c-jis/standards-alignment.md) — W3C/JIS/VC alignment points.
- [`glossary.md`](glossary.md) — shared vocabulary (`aint`, `raint`, `waint`, SNAFT, MUX, TIBET, can_carry).

## Quick reading path

1. [`architecture/boundary-map.md`](architecture/boundary-map.md)
2. [`test-vectors/valo-pass-degrade-halt.md`](test-vectors/valo-pass-degrade-halt.md)
3. [`evidence/provision-status-samples/local-only-codex-denied.json`](evidence/provision-status-samples/local-only-codex-denied.json)
4. [`evidence/launch-verdict-samples/not-ready-egress-denied.json`](evidence/launch-verdict-samples/not-ready-egress-denied.json)
5. [`w3c-jis/standards-alignment.md`](w3c-jis/standards-alignment.md)

## Core comparison — PASS / DEGRADE / HALT

A deterministic gate before execution (VALO-style) returns PASS / DEGRADE / HALT. IAB provides the
**local substrate facts** such a gate needs, and enforces the same shape natively:

| Gate language | IAB verdict | Meaning |
|---|---|---|
| PASS | `0x4000`, `can_carry=true` | action may proceed |
| DEGRADE | triage hold · partial posture · degraded sensor | action needs review or a lower claim |
| HALT | `0x0000:<reason>` | action refused **before** execution starts |

The gate should not be fed by prompt promises. It should be fed by local facts — and the verdict is
enforced by the runtime, not by the model promising to behave.

## The boundary — what the substrate does NOT own

- **The substrate owns** (Layer 0–2): identity, authority, the posture floor, monotonic admissibility,
  native fail-closed enforcement, and receipts.
- **It deliberately does NOT own** (Layer 3): enterprise policy, compliance overlays, vertical
  workflows, or an external clearance / governance layer. Those sit **above** it and *feed* the gate —
  they should never need to rebuild Layer 0, and they never become the root of authority for the box.
- "Execution allowed" always means *allowed by the current local substrate facts*, never merely
  *approved by an external service*.

## The wider ecosystem — where this fits

The pieces are deliberately open and spread across the commons:

- **[ainternet.org](https://ainternet.org)** — the network itself: `.aint` identity + discovery (AINS),
  the readable architecture overview, and the live protocol drafts.
- **[github.com/ainternet-org](https://github.com/ainternet-org)** — the commons org: this dossier + the
  conformance families. The substrate implementation source opens publicly on release.
- **JIS · TIBET · AINS** — the identity, provenance and discovery layers, submitted as IETF
  Internet-Drafts and aligned to W3C VC/DID (see [`w3c-jis/standards-alignment.md`](w3c-jis/standards-alignment.md)
  and ainternet.org).
- **[github.com/Jtel-ZTIP-w3c](https://github.com/Jtel-ZTIP-w3c)** — ZTIP (Zero-Trust Identification
  Protocol) + the runnable interop conformance family (comms · security · evidence · VINK attestation):
  independent implementations agree, **bring your own verifier** — no vendor needed to prove it green.
- **[Humotica](https://humotica.com)** — the steward / keeper: the entity that signs, supports and
  carries the substrate for production and enterprise deployments. The commons stays open (`pip install`,
  build your own); the keeper is the accountable, supported path for scale.

## Reviewing further

If this dossier is the right level, scoped access to the implementation repo or a live walk-through of
a running box can follow under an agreed scope. The separation described here is **running**, not
theoretical — it boots, it seals, and it refuses.
