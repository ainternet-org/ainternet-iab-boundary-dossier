# Layer Map

IAB separates runtime, proof and operation.

```text
L3  Operate       cockpit, agents, lanes, comms
L2  Prove         BOMs, receipts, conformance, reports
L1  Run           identity, posture, MUX, SNAFT, continuity, carrier floor
L0  Host Floor    KVM, TPM, network, memory, hardware facts
```

## L0 Host Floor

The physical and kernel-level facts:

- KVM;
- TPM;
- entropy;
- network interfaces;
- RAM/swap pressure;
- GPU presence when relevant;
- memfd/CRUST capability.

## L1 Run

The box must be able to boot, refuse, route, reseed and close with proof.

Examples:

- root identity;
- `raint` spawn;
- actor grants;
- SNAFT posture;
- MUX surfaces;
- `.tza` carrier floor;
- continuity boot;
- `0x0000:<reason>` refusal.

## L2 Prove

The box explains and verifies itself:

- System-BOM;
- RAM-BOM;
- MUX-BOM;
- Role-BOM;
- static SBOM/CBOM;
- TIBET-BOM;
- release signatures;
- conformance vectors.

L2 may degrade visibly. Missing proof tooling should not be disguised as full
posture.

## L3 Operate

Humans and actors use the box through:

- CLI;
- TUI;
- cockpit/web console;
- lanes;
- cmail;
- ipoll;
- actor workspaces;
- runtime-bound PTY.

L3 must not invent authority. It is a lens over the same facts and verbs.

## Placement Rule

Ask:

```text
Does this component make the box run, prove itself, or get used?
```

Then place it:

| Answer | Layer |
|---|---|
| run | L1 |
| prove | L2 |
| use | L3 |
| physical fact | L0 |

