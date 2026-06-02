# Server reference

`joule-edge-server` exposes the cascade over HTTP with an Anthropic-compatible wire format. Drop-in for any client that already speaks `/v1/messages`.

## Start

```bash
joule-edge-server --bind 127.0.0.1:7777
```

Flags:

- `--bind <addr>` — listen address (default `127.0.0.1:7777`)
- `--budget-joules <n>` — per-request hard cap; the cascade refuses to escalate past this
- `--router <rule|ml|hybrid>` — routing policy (default `hybrid`)
- `--verify <first|all|none>` — verifier-first (default), Boltzmann posterior, or off
- `--data <path>` — scar-memory and history persistence directory
- `--launchd` / `--systemd` — install as a service

## Endpoints

| Method | Path | Body |
|---|---|---|
| `POST` | `/v1/messages` | Anthropic-format request, including `model`, `messages`, optional `max_tokens` |
| `GET` | `/v1/models` | Available cascade variants |
| `GET` | `/health` | Liveness / readiness |
| `GET` | `/metrics` | Prometheus metrics |
| `GET` | `/cascade/explain?id=<request_id>` | Per-tier walk for a recent request |

## Response shape

```json
{
  "id": "msg_…",
  "type": "message",
  "role": "assistant",
  "content": [{ "type": "text", "text": "12" }],
  "model": "pattern-lang",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 9,
    "output_tokens": 1,
    "joules_spent": 0.0000041,
    "tier_used": "L1",
    "verifier": "recompute",
    "verdict": "accept"
  }
}
```

## What gets routed where

- Arithmetic and deterministic transforms — L1 (lawful), verified by recompute oracle or Lean proof
- Pattern-matched code synthesis — L1, verified by typecheck
- Tabular / retrieval queries — L2 (MRL embedding) gated by DeBERTa entailment
- Open-ended text — L3 (Gemma 4) gated by k-of-n vote
- Anything above the request budget — refused with a structured error, not silently downgraded

## Integration tips

Any client library that targets the Anthropic Messages API works without modification. The extra fields in `usage` are additive — clients that ignore them keep working.
