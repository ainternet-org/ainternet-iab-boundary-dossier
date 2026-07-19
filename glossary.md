# Glossary

## `aint`

An identity name in the AInternet namespace, such as `jasper.aint` or
`qwen.aint`.

## `raint`

A runtime node instance. A root can spawn/bind actors under a `raint`.

## `waint`

A surface or runtime-bound actor projection, such as `codex.waint` or
`cockpit.waint`.

## SNAFT

The posture/firewall layer. It determines which surfaces and egress routes can
exist under the current posture.

## MUX

The route and surface multiplexer. It maps which actor can reach which surface
or lane.

## TIBET

The receipt and continuity chain. Actions, refusals and lifecycle transitions
are recorded as hash-linked ticks.

## `can_carry`

The folded decision that an actor/action can proceed under current authority,
posture, locality and machine facts.

## Dark Reason

A fail-closed refusal reason, usually encoded as `0x0000:<reason>`. Absence is
evidence; missing facts do not become implicit permission.

## Locality

Where execution traffic goes, derived from actual route/egress facts, not from
where a binary lives. A CLI binary can be local while its egress is external.

