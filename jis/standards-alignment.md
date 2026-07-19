# Standards Alignment: W3C, JIS, TIBET

Status: initial public-safe alignment note.

## W3C Verifiable Credentials

IAB actor and root grants can be projected toward W3C Verifiable Credential
shapes:

- issuer: human/root identity;
- subject: actor identity;
- claims: surfaces, posture, ttl, scope;
- proof: signature over grant payload;
- status: active, voided, tombstoned, revoked.

The local box should not depend on a remote VC resolver to enforce authority.
VC alignment is an interoperability projection over local facts.

## W3C DID / Identity

`aint` names can map toward DID-style identifiers, but the box still needs a
local root-of-trust:

```text
did/aint projection -> useful for exchange
local root key       -> required for native enforcement
```

## JIS

JIS is the relation/handshake layer for finding and proving a peer before
surfaces materialize.

Alignment points:

- relation-first access;
- no session, no surface;
- handshake receipt before privileged route;
- local-first operation through NAT/CGNAT when possible;
- dark by default from outside.

## TIBET

TIBET is the causal receipt chain.

It supplies:

- event ordering;
- prev/self hash linkage;
- proof of refusal as well as proof of execution;
- continuity across close, crash, reseed and resume.

For external governance systems, TIBET is the part that makes an execution
decision auditable after the fact without relying on model memory.

## Humotica / IAB Separation

Humotica may provide stewardship, enterprise support, policy profiles and
integration work.

AInternet-in-a-box remains the substrate:

- local-first;
- open primitives;
- dark by default;
- sovereign enforcement;
- receipts as proof.

That separation lets companies build governance and applications on top without
replacing the floor.

