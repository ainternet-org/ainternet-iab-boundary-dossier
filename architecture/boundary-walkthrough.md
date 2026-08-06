# IAB ↔ external clearance — a concrete boundary walkthrough

*For external reviewers and integrators. (Originally written as a boundary walkthrough for Njål Gaute Solland, VALO Research Group / REHT Protocol.)*
*Author: the AInternet-in-a-Box substrate team. Status: dev-grade, but every substrate output below is verbatim from a running box — not described, run.*

---

## 0. The two-verdict model (why neither replaces the other)

You summarised it precisely, so we adopt your framing verbatim:

- **Substrate (IAB) answers:** *can **this actor** carry **this exact action** under the local
  identity × mandate × posture × runtime × locality × machine-facts currently available?*
- **Clearance (VALO / REHT) answers:** ***should** the consequential action execute under its purpose ×
  organizational authority × enterprise policy × evidence × expected consequences × current governance
  state?*

Two invariants hold the boundary:

1. **A substrate DENY is final.** No clearance verdict can lift it.
2. **A substrate PASS is not, by itself, legitimacy.** It only says the local floor permits the mechanics.

That is exactly why an independent execution-clearance layer plugs in *above* the substrate without
rebuilding Layer 0. The substrate is deliberately *narrow*: it owns identity, posture, admissibility,
fail-closed execution and receipts — not the enterprise policy world above it.

---

## 1. Verdict scope — read this first (the ambiguity you flagged)

You caught the sharpest integration risk: *"top-level `result` is `0x4000` while the actor
`carry_decision` is `deny` — the scope needs to be unambiguous for machine integration."* Correct, and
intentional. The two are **different verdicts with different scopes**, and a machine must never conflate
them.

| field | scope | means | a machine keys on it for |
|---|---|---|---|
| `result: 0x4000` | **the query / operation** | this *readout* is a valid, well-formed, sealed answer | "did the status call succeed?" — **never** "is anything permitted?" |
| `carry_decision` | **this actor × this action** | admissibility of one bound action right now | the actual allow / deny / needs_* gate |
| `launch.ready` | **the box as a whole** | fail-closed launch verdict for going live | "may the sealed pool boot?" |

**Real output, verbatim, from a running dev box** (`./box provision status --json`, trimmed):

```json
{
  "result": "0x4000",
  "sealed": false,
  "authentic": false,
  "ready": "0/1",
  "can_carry": [
    { "aint": "shania-tw.aint", "carry_decision": "allow", "bindable": true, "why": ["host-cannot-carry"] }
  ],
  "fold": { "ready_to_go_dark": false, "blocking": ["ai_sbom", "host_can_carry", "missing:machine_posture"] }
}
```

Note: `result: 0x4000` **while** `sealed: false`, `ready: 0/1`, and `blocking` non-empty. The `0x4000`
means *"here is a valid, signed answer"* — it says nothing about permission. The admissibility verdict is
`can_carry[].carry_decision`; the boot verdict is `ready`. In a machine adapter you would bind on
`carry_decision` (per action) and `launch.ready` (per boot), and treat `result` purely as a transport /
integrity code. We will make this scope explicit in `--dev` and the TUI so a human reads the same thing a
machine does.

`0x4000` = sealed/valid · `0x0000:<reason>` = a refusal or an open stance, always with a reason suffix.
Same code family, but the **suffix carries the meaning** — which is exactly why a human-readable copy layer
is on the roadmap (so you read the intent, not decode the hex).

---

## 2. The one concrete action we trace

Everything below follows a single action end to end:

> **actor** `shania-tw.aint` (runtime: **codex-cli**), acting **on-behalf-of** root `jasper.aint`,
> **proposes egress** to `api.openai.com` (locality: **external**).

---

## 3. Local substrate facts → the admissibility fold

`carry_decision` is not a flag; it is a `min()` over three flanks, computed fresh each read:

```
carry_decision = f( posture-floor(reach ceiling)  ×  action-locality  ×  per-actor grant )
```

Same actor, same action, four states — **all real** (from `posture.carry_decision`, the exact function
the box uses):

| # | floor (posture) | egress grant for api.openai.com | `carry_decision` | scope note |
|---|---|---|---|---|
| A | `LOCAL_ONLY` (dark) | — | **`deny`** | floor is the ceiling; external reach is below it → **final** |
| B | `NORMAL` | — | **`needs_egress_grant`** | floor permits, but no per-action grant covers this target |
| C | `NORMAL` | ✓ (`api.openai.com`) | **`allow`** | substrate PASS — mechanics permitted (**not** legitimacy) |
| D | `LOCAL_ONLY` (revoked mid-run) | ✓ (still present) | **`deny`** | see §6 — the floor drops, deny wins over a standing grant |

Reading of the scopes: **A is a final substrate denial.** **C is a substrate PASS** — the point at which
your clearance verdict becomes decisive. A grant only ever *opens above the floor*; it can never reach
below it (row D).

---

## 4. External clearance input — where VALO binds (illustrative; your side)

The substrate emits, per bound action, a stable **join key**: the grant's canonical hash `grant.self`
(see §5). A clearance verdict is *bound to the exact action* by referencing that key plus the concrete
(actor, verb, target). Illustrative shape (this is your layer, not ours — shown only so the correlation is
concrete):

```json
{
  "kind": "org.valo.clearance-verdict.v1",
  "binds": {
    "actor": "shania-tw.aint",
    "verb": "egress",
    "target": "api.openai.com",
    "substrate_grant": "sha256:8345956d1997b65f9f656a1fdb8d528de3c5a6b84c85cf06c09f140be0f118a2"
  },
  "clearance": "allow",
  "reason": "purpose+policy+evidence satisfied",
  "expires_at": 1784978200,
  "self": "sha256:<valo-computed>"
}
```

The binding is to `substrate_grant` (a content hash of the *exact* grant this action runs under) — not to
the actor in general. If the substrate re-grants (a new posture, a new causal position → a new
`grant.self`), a clearance bound to the old grant no longer matches. Drift is therefore detectable by hash
mismatch, not by trust.

---

## 5. Execution path → the substrate's own receipts (real)

Under `carry_decision: allow`, `./box bind` writes the grant **before** any process starts, spawns the
runtime behind a PTY **sandboxed by default** (bwrap: box-fs read-only, per-session overlay the only
writable place, network off unless a net-surface is granted), and seals on exit.

**Real `grant.json`** (execution-authority receipt, verbatim):

```json
{
  "kind": "org.ainternet.tui.cli-session-grant.v1",
  "actor": "shania-tw.waint",
  "surfaces": ["pty.aint", "audit.aint", "tool.local-shell.waint", "net.egress.waint"],
  "runtime_provider": "codex",
  "egress_target": "api.openai.com",
  "locality": "external",
  "carry_decision": "allow",
  "fold": {
    "root": "jasper.aint", "role": "runtime-actor", "grant": "cli-session",
    "posture": "NORMAL", "locality": "external", "carry_decision": "allow",
    "causal_position": 6, "floor": "seal-required"
  },
  "session_id": "shania-tw_waint-1784977748-6",
  "self": "sha256:8345956d1997b65f9f656a1fdb8d528de3c5a6b84c85cf06c09f140be0f118a2"
}
```

`net.egress.waint` is present in `surfaces` **only because** the action's locality is `external` and the
grant covers it — the sandbox permits network iff that surface is present. A loopback/local actor's grant
carries no net-surface (dark egress). Enforcement is native: the fail-closed egress gate refuses to spawn
if `carry_decision != allow`, *before* any process runs.

**Real `seal.json`** (seal-on-quit receipt, verbatim) — written on a clean `/quit`:

```json
{
  "kind": "org.ainternet.tui.cli-session-seal.v1",
  "actor": "shania-tw.waint",
  "session_id": "shania-tw_waint-1784977748-6",
  "event": "actor-exit-clean",
  "closed": true,
  "graceful": true,
  "exit_code": 0,
  "grant_hash": "sha256:2d30d9faa4ecfd5422fd60d3d4e52f2fd0b1b02952d7d501a6a16d03553292ee",
  "causal_seq": 8,
  "self": "sha256:d338cc36ccd5c73bf9a7499a68db775614376995802cfd24e1475584a7489b9a"
}
```

A clean `/quit` seals `closed: true` (a closed stance with proof). A crash / kill / non-zero exit leaves
**no closed stance** — the session stays unsealed and the *next* bind must reattest (reseed) before it may
run. There are no sleeping sessions.

---

## 6. State drift, expiry, revocation during runtime

Two independent revocation mechanics, both fail-closed:

**(a) Posture drift — the floor is the ceiling.** Row D above is the real case: with the egress grant
*still present*, dropping the floor to `LOCAL_ONLY` mid-runtime flips `carry_decision` back to `deny`.
A grant opens *above* the floor; it can never carry an action *below* it. Because `carry_decision` is
recomputed each read (not cached), the next admissibility check denies — the standing grant does not save
it.

**(b) No-closed-stance — crash/kill.** If the runtime dies without a clean seal, the grant is left with no
matching seal → the next bind refuses with `0x0000:reseed-required` (recovery: an auditable `actor-reseed`
receipt closes the open stance before a fresh bind). Expiry on the clearance side (`expires_at`) is your
layer's equivalent — a stale clearance simply no longer matches a fresh `grant.self`.

The composite rule: **execution requires (substrate `allow`) AND (clearance `allow`), evaluated at the
moment of action; either side going stale or dropping denies, and a substrate deny is final.**

---

## 7. The final execution verdict (decision table)

| substrate `carry_decision` | clearance verdict | **execute?** | why |
|---|---|---|---|
| `deny` | *any* | **NO** | substrate denial is final |
| `needs_egress_grant` | *any* | **NO** | not yet admissible; resolve the grant first |
| `allow` | `deny` / expired / missing | **NO** | mechanics permitted, but not legitimate |
| `allow` | `allow` (bound to this `grant.self`) | **YES** | both verdicts agree, on the exact action |

The substrate never asks *why*; the clearance layer never re-checks *whether the mechanics are locally
possible*. Two questions, two owners, one action.

---

## 8. Receipt correlation — the two chains, verified

The substrate keeps two linked structures, both real above:

- **Session receipts:** `grant.json` → `seal.json`. The seal binds to the grant by **shared
  `session_id`** and by **`grant_hash` = a content hash of the grant file** (tamper-evident).
- **Ledger:** a hash-chained tibet tick log; each tick carries `prev → self`. Real excerpt (this exact
  action's tail):

  ```
  egress.grant        prev 1312fdbf4640  self 725bdc8c88d7
  cli-session.grant   prev 1387368042dd  self c1b160824cf5
  cli-session.sandbox prev 2a531d11b067  self f9e130feed81
  cli-session.seal    prev 7d9c35a12870  self c91477139631
  ```

**Correlation, verified on the box (not asserted):**

```
seal.grant_hash            = sha256:2d30d9faa4ecfd5422fd60d3d4e52f2fd0b1b02952d7d501a6a16d03553292ee
sha256(grant.json file)    = sha256:2d30d9faa4ecfd5422fd60d3d4e52f2fd0b1b02952d7d501a6a16d03553292ee   ← MATCH
grant.self (canonical JSON)= sha256:8345956d1997b65f9f656a1fdb8d528de3c5a6b84c85cf06c09f140be0f118a2   ← the VALO join key
shared session_id          = shania-tw_waint-1784977748-6
```

So the two receipt chains correlate at a single, hash-stable point: **`grant.self`** is the join key your
clearance verdict binds to (§4); **`grant_hash` + `session_id`** tie the substrate's own grant↔seal pair;
the **tibet chain** orders everything causally. A VALO receipt that references `grant.self` is provably
about *this action, this grant, this run* — and drifts detectably the moment the substrate re-grants.

---

## 9. Real vs illustrative (so nothing is oversold)

- **Real, verbatim from a running dev box:** `provision status` (§1), `launch` verdict (§10), the four
  `carry_decision` states (§3), `grant.json` + `seal.json` (§5), the `grant_hash` correlation and tibet
  chain (§8).
- **Illustrative (your side):** the VALO clearance-verdict shape (§4) and its `expires_at`. We shaped it
  only to make the join concrete; the actual clearance semantics are yours.

It is dev-grade. But the separation is *running*, not described.

---

## 10. Where the boundary sits — the plug-in point

```
                 SHOULD it execute?  (purpose · org-authority · policy · evidence · governance)
   VALO / REHT  ─────────────────────────────────────────────────────────────────────  clearance verdict
        │  binds to grant.self (§4)                                    ▲ final verdict = substrate AND clearance
        ▼                                                              │
   ─────────────────────────────────────────────────────────────────────────────────
   IAB substrate │  CAN this actor carry this exact action?  (identity · mandate · posture · runtime · locality · machine)
                 │  fail-closed execution (bwrap) · grant→seal receipts · hash-chained ledger
                 │  carry_decision  →  bind  →  seal      (deny is FINAL; PASS ≠ legitimacy)
```

**Substrate owns:** identity, posture, admissibility (`carry_decision`), fail-closed execution, receipts.
**Clearance owns:** purpose, organizational authority, policy, expected consequences, governance state.
**The adapter** (the small shared piece worth building): clearance consumes the substrate's `carry`-fold +
`grant.self`, emits a clearance verdict bound to that `grant.self`, and execution requires both — with the
substrate deny remaining final.

For the launch gate itself, the substrate is honest about *not being ready* the same way:

```json
{ "ready": false, "reason": "not-sealed",
  "why": "nothing sealed — compose the pool, then: sudo ./box provision seal",
  "next": "sudo ./box provision seal" }
```

Fail-closed by default, with the reason and the next step in the same breath.

---

*Next, if useful: a small adapter spec + a shared conformance vector (one action, the four carry states, a
revocation, and a correlated receipt pair) that both sides can test against. A live screen-share of
provision → launch → bind → runtime-revocation → receipt is easy to arrange from here.*
