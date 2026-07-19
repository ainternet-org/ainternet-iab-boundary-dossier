# Boundary Map

## One Sentence

AInternet-in-a-box is the local substrate that separates AI reasoning from
execution by making every action pass through identity, posture, admissibility,
native enforcement and receipts.

## The Execution Boundary

```text
AI / operator intent
    |
    v
proposal: "do X as actor Y"
    |
    v
local fact fold
  - root identity
  - actor mandate
  - runtime binding
  - locality / egress
  - SNAFT posture
  - MUX surfaces
  - System-BOM
  - continuity state
  - open / sealed sessions
    |
    v
admissibility verdict
  - allow: 0x4000
  - degrade: triage / partial / hold
  - block: 0x0000:<reason>
    |
    v
guarded execution
    |
    v
TIBET receipt
```

The AI can produce intent, arguments, plans, code and messages. It does not
acquire authority by producing them.

## What The Substrate Owns

IAB owns the lower floor:

- identity root and actor mandate;
- runtime binding and actor locality;
- monotonic posture and SNAFT floor;
- MUX/surface visibility;
- fail-closed action gating;
- sealed runtime and clean-exit evidence;
- TIBET receipts and continuity.

The substrate answers:

```text
Can this actor carry this action under the facts visible now?
```

## What Higher Layers Own

Product, governance, policy and enterprise layers own:

- business-specific clearance profiles;
- workflow approvals;
- compliance rules;
- customer-specific risk models;
- domain policy vocabulary;
- reporting formats.

Those layers may feed the gate. They must not bypass the local substrate.

## Boundary Rule

External clearance can contribute policy facts, but the box remains sovereign
over native enforcement and receipts.

An external service may say "this action should be allowed". The local box still
decides whether it can carry.

## VALO / REHT Comparison

Public VALO language describes an L1 Guardian, a deterministic coherence gate,
and PASS / DEGRADE / HALT before execution. That maps cleanly to IAB:

| VALO-style term | IAB primitive |
|---|---|
| L1 Guardian | local admissibility fold + SNAFT/MUX enforcement |
| PASS | `0x4000`, launch ready, can carry |
| DEGRADE | triage, partial System-BOM, degraded carry |
| HALT | `0x0000:<dark-reason>` before spawn |
| immutable audit trail | TIBET receipt chain |
| governance before consequence | audit/admissibility before execution |

The strongest integration point is "what feeds the deterministic gate": IAB
feeds it local substrate facts.

