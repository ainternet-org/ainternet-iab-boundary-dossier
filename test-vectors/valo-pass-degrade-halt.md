# Test Vectors: PASS / DEGRADE / HALT

These vectors map VALO-style gate language to IAB runtime outcomes.

## V1 PASS: Local Actor Can Carry

Input facts:

- actor: `qwen.aint`;
- provider: `ollama`;
- endpoint: `http://127.0.0.1:11434`;
- posture: `LOCAL_ONLY`;
- model selected;
- root identity present;
- actor grant present.

Expected verdict:

```json
{
  "result": "0x4000",
  "actor": "qwen.aint",
  "locality": "local",
  "can_carry": true
}
```

Meaning: PASS.

## V2 HALT: External CLI Under LOCAL_ONLY

Input facts:

- actor: `codex.aint`;
- runtime: `codex`;
- runtime binary: local;
- egress target: `api.openai.com`;
- posture: `LOCAL_ONLY`;
- egress grant: absent or not permitted by posture.

Expected verdict:

```json
{
  "result": "0x0000:egress-not-permitted",
  "actor": "codex.waint",
  "locality": "external",
  "carry_decision": "deny"
}
```

Meaning: HALT before spawn.

## V3 DEGRADE: Partial System-BOM

Input facts:

- System-BOM sensors present: static, hw, mux, net;
- missing or partial: role, gpu, ram;
- TIBET chain intact.

Expected verdict:

```json
{
  "system_bom": {
    "verdict": "partial",
    "present": "4/7"
  },
  "ledger": {
    "verdict": "intact"
  }
}
```

Meaning: DEGRADE. The evidence chain can be intact while the physical/system
claim is partial.

## V4 HALT: Launch Not Ready

Input facts:

- provision state exists;
- can_carry drift or missing posture;
- launch requested.

Expected verdict:

```json
{
  "ready": false,
  "result": "0x0000:launch-not-ready",
  "next": "sudo ./box provision seal"
}
```

Meaning: HALT. The launch button or command must fail closed.

## V5 DEGRADE/PAUSE: Open Session Requires Reseed

Input facts:

- actor session was opened;
- no clean seal exists;
- new bind requested.

Expected verdict:

```json
{
  "result": "0x0000:reseed-required",
  "unsealed_sessions": ["codex_waint-..."],
  "next": "sudo ./box bind --reseed ..."
}
```

Meaning: PAUSE/HALT until continuity is repaired.

