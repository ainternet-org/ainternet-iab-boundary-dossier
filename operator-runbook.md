# Operator Runbook

Status: public-safe command map.

The CLI is the contract. TUI and cockpit are lenses over these verbs.

## Fresh Human Root

```sh
./box up
./box enroll jasper.aint --human
```

After a human root is enrolled, the next boot materializes that identity.

## Local API Actor

Example: local Ollama on loopback.

```sh
./box actor onboard qwen.aint --kind api --provider ollama --endpoint http://127.0.0.1:11434
./box actor test qwen.aint
./box actor cli qwen.aint
```

Local API actors need endpoint and model. They do not need an internet egress
grant when the endpoint is loopback.

## LAN API Actor

Example: Ollama on another machine in a private LAN.

```sh
./box actor onboard gemma.aint --kind api --provider ollama --endpoint http://192.168.x.x:11434 --model <model>
./box grant egress gemma.aint --to <lan-host>
./box actor test gemma.aint
```

Private LAN is not loopback. Treat it as relation/grant work, not automatic
local authority.

## CLI Actor With External Egress

Example: Codex CLI. The binary is local, but the runtime egress is external.

```sh
./box actor new codex.aint --kind codex --alias "Codex" --ai
./box pop --agent codex.aint
./box runtime prefetch codex
./box runtime stage codex
./box runtime switch codex <version>
./box provision set-snaft NORMAL
./box grant egress codex.aint
./box bind codex
```

Exit with `/quit` or Ctrl-D. Clean exit writes seal evidence.

## Launch Gate

```sh
./box provision status --json
./box launch --json
./box launch
```

`box launch` must not boot through a false green. It first checks the folded
verdict.

