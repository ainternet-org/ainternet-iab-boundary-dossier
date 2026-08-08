# VALO ⇄ IAB Boundary Profile — first cut (IAB side)

Date: 2026-08-08 · Status: **first cut, IAB side.** Drafted for VALO to map against REHT semantics and mark
where the contracts genuinely differ. Deliberately narrow — a small profile that is actually finished beats a
broad one that is half-done. Forcing function: Agentic Internet Workshop, Mountain View, November 2026.

This is a **boundary, not an integration.** IAB is not a required component inside VALO and VALO is not a
dependency of IAB. Neither side inherits the other's release cycle or internal semantics. The two are joined by
one external binding reference, so that a third runtime — neither VALO nor IAB — can bind to the same contract
without either of us being involved. That independence is the point.

## The invariant (top of the profile)

> **Authorization is not a durable truth about an action. It is a claim that only becomes valid again if the
> executing side can re-establish, at the moment of consequence, that the action, the authority, and the relevant
> state are still the same as what was approved.**

An authorization does not travel as a verdict that stays true. It travels as a **binding to a specific action and
execution state** — the *execution envelope*. At execution the executing side re-derives that envelope
independently and compares.

- **Exact match →** evaluation may continue.
- **Any mismatch →** hard deny.

The executing side never asks *"was this approved?"* It asks *"is this still the thing that was approved?"*

Validity is therefore not a property of the authorization at all. It is a property of the **moment of execution**,
re-established every time. Anything else is a verdict with an expiry date nobody can see.

If this invariant holds, the negative conformance cases below are not special cases — they are simply the distinct
ways the re-derived envelope fails to match the thing that was approved.

## Causal, not temporal

The comparison gets its teeth from ordering, and the ordering is **causal, not wall-clock.**

Two independently owned systems have two clocks. `valid until 14:03` is a claim that depends on NTP — an authority
*outside* the boundary. That is exactly the hidden dependency the boundary exists to remove. So the profile does
not ask *"when did this happen?"* It asks *"what provably came before what?"* — specifically:

> At execution: **is there anything between the authorization and this action that should not be there?**

Both sides can answer that without a shared clock. This collapses three of the five cases into one shape:

- **stale authority** = *something intervened*
- **replay** = *this position was already used*
- **drift** = *the state it bound to is not the state that is here*

**expired scope** and **receipt continuity** stay separate.

Prior published work underlies this: **RS-2026-001 — Causal Substrate Audit: Lamport-Anchored Evidence Under
Time-Source Asymmetry** (Red Specter × Humotica joint paper, Zenodo `10.5281/zenodo.20338260`; a signed
Ed25519-per-block bundle is available for byte-identical reproduction across mirrors). The semantics here are not
invented on the spot.

## Ownership lines

| Owned by **VALO** | Owned by **IAB** |
|---|---|
| the execution envelope | local capability |
| the authorization semantics | posture |
| the REHT verdict | enforcement |
| the binding reference `execution_envelope_hash` | receipts |

The authorization binds to a **VALO-owned `execution_envelope_hash`**. IAB re-derives its side of the envelope at
presentation and enforces the comparison locally; VALO owns what the envelope *means*. Because the binding is an
external reference rather than an embedded component, a third runtime binds the same `execution_envelope_hash`
contract without either owner in the loop.

## The conformance vector — what MUST be refused

A profile that only asserts what must be *accepted* tests one side's output format, not the boundary. The boundary
is defined by its **negative** cases: what each side must hard-deny. The vector runs in **both directions** — each
case specifies what a side guarantees at presentation, what the other side is entitled to rely on, and the
condition that invalidates that reliance.

For each case: an authorization that presents cleanly by every *syntactic* measure must still be **hard-denied**
when the re-derived envelope no longer matches. "Presented" is never "valid."

### 1. Drift
The envelope bound to an action or execution state; that state has since changed.
- **Must refuse:** any action whose re-derived state differs from the state the envelope bound to.
- **IAB establishes it by:** re-deriving the execution envelope at presentation and comparing to the bound value;
  a difference is a mismatch, not a warning.

### 2. Expired scope
The capability was presented after its scope no longer covers the requested action.
- **Must refuse:** any action outside the capability's live scope at the moment of presentation.
- **IAB establishes it by:** evaluating `grant.self` scope **at presentation time**, not at grant time.

### 3. Broken receipt continuity
The evidence chain from authorization to this action has a gap.
- **Must refuse:** any action whose receipt chain cannot be reconstructed as an unbroken, verifiable sequence.
- **IAB establishes it by:** verify-on-read of the causal chain (each entry hash-links its predecessor); a gap
  forces safe-mode, never a rendered "history."

### 4. Stale authority
An authority was superseded, revoked, or tombstoned between authorization and execution.
- **Must refuse:** any action whose authority has been intervened upon since the authorization.
- **IAB establishes it by:** a freshness verdict against the tombstone ledger — *something intervened* → deny.
  Freshness is checked, not assumed from a syntactically valid identity.

### 5. Replay
A previously used authorization/position is presented again.
- **Must refuse:** any action at a position that was already consumed.
- **IAB establishes it by:** the tombstone/position ledger — *this position was already used* → deny. Exact
  resurrection and indirect reuse both fail the causal comparison.

## What IAB guarantees at presentation

- The capability presented is evaluated for **live scope** at the moment of presentation (`grant.self`), not for
  scope that was live at grant time.
- The execution envelope is **re-derived independently** on the IAB side; IAB does not trust a carried verdict.
- Every enforcement decision emits a **receipt** into a verify-on-read causal chain, so the *"what came before
  what"* question is answerable after the fact by either side.
- Ordering claims are **causal**; IAB introduces no wall-clock dependency into the boundary.

## Scope note

Five negative cases, one invariant, one binding reference. Nothing here couples either roadmap to the other.
Open questions for VALO to mark up: which of these become REHT invariants, which belong in the interoperability
profile, and which stay external substrate rather than a VALO dependency (TIBET itself must **not** become a VALO
dependency — only the contract crosses the line). The disagreements are the interesting part.
