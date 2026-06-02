# Pattern-lang

**The pattern catalog and edge-first AI cascade runtime.**

Pattern-lang composes two layers:

1. **A lawful synthesizer** — 690 named primitives, 41 target languages, deterministic codegen. Intent maps to a typed DAG of patterns; the DAG lowers to Cortex IR; the IR emits in any target. Same input, same output. Picojoule-budgeted by construction.
2. **A verifier-first cascade runtime** — L0 cache → L1 lawful → L2 small models (Prism MLGRU, Liquid CfC, MRL, DeBERTa) → L3 medium models (Gemma 4, LMM) → L4 frontier. Each tier wrapped in an independent verifier (recompute, EBM constraint, Lean proof, k-of-n vote). Accept on agreement, escalate on reject.

**Capability per joule, not capability per parameter.**

This is the **public release surface**. Source is private under [`openIE-dev/pattern-lang-core`](https://github.com/openIE-dev/pattern-lang-core). Use the binaries.

## Status

> This repository ships **release binaries and documentation**, not source. Pattern-lang is **free to use software**, not an open-source project. See [LICENSE](./LICENSE) for the Business Source License 1.1 terms — converts to Apache-2.0 four years after each binary's release date.

Releases are attached to [GitHub Releases](https://github.com/openIE-dev/pattern-lang/releases). The marketing surface is at [pattern-lang.ai](https://pattern-lang.ai).

## What you get

- `pattern-lang` — CLI for synthesis, composition, and the cascade
- `joule-edge-server` — Anthropic-compatible HTTP server (`POST /v1/messages`) with `joules_spent` and `tier_used` in usage
- `joule-edge-cli` — Plan + Execute + Diagnose + Compose wired into a single binary

Target platforms:

| Platform | Status |
|---|---|
| macOS arm64 (Apple Silicon) | shipping |
| Linux x86_64 (distroless, ~5 MB binary) | shipping |
| Linux arm64 (Raspberry Pi 4/5) | shipping |
| Android arm64-v8a | shipping |
| WASM | tracking |

## Install

Once a release is published:

```bash
# macOS / Linux — grab the latest release for your platform
curl -fsSL https://github.com/openIE-dev/pattern-lang/releases/latest/download/install.sh | sh

# Verify
pattern-lang --version
```

For air-gapped installs, download the binary tarball from the [Releases](https://github.com/openIE-dev/pattern-lang/releases) page and place it on `$PATH`.

## Quick taste

### CLI

```bash
# Engage the catalog interactively
pattern-lang engage

# Compose a typed DAG for an intent
pattern-lang compose "sort a list and remove duplicates"

# Emit in your target language
pattern-lang emit --target rust
pattern-lang emit --target python
pattern-lang emit --target go
```

### HTTP endpoint (Anthropic-compatible)

```bash
# Start the local edge server
joule-edge-server --bind 127.0.0.1:7777

# Hit it with any Anthropic-format client
curl http://127.0.0.1:7777/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "pattern-lang",
    "messages": [{"role": "user", "content": "gcd of 12 and 8"}]
  }'

# Response includes usage.joules_spent and usage.tier_used
```

### Embedded library

For Rust users, the public crate ships as binary releases only; embedded library integration is in [`openIE-dev/pattern-lang-core`](https://github.com/openIE-dev/pattern-lang-core) (private; contact for access).

## How the cascade walks

```
query →  L0 cache              (~nJ)  miss → continue
      →  L1 lawful synth       (~µJ)  accept if catalog matches
      ↓  verify (recompute / Lean / EBM)
      →  L2 Prism / Liquid     (~µJ–mJ)
      ↓  verify (k-of-n)
      →  L3 Gemma 4 / LMM      (~mJ)
      ↓  verify
      →  L4 frontier           (~10s mJ)  RPC, deferred
```

Each tier reports its joule estimate. The router walks tiers in cost order; the verifier gates each step. On reject, escalate. On accept, return — with a receipt.

Live verification of the cost wedge:

```bash
# After install:
pattern-lang bench cascade

# Shows: verify_first is 2.86× cheaper than always-frontier
# at the same 100% correctness on the in-tree benchmark.
```

## Documentation

- [`docs/getting-started.md`](./docs/getting-started.md) — install, first query, configuration
- [`docs/cli.md`](./docs/cli.md) — CLI reference
- [`docs/server.md`](./docs/server.md) — joule-edge-server and the `/v1/messages` API
- [`docs/cascade.md`](./docs/cascade.md) — tier model, routing policies, verifier types
- [`docs/license-faq.md`](./docs/license-faq.md) — BSL 1.1 in plain English

The deeper architecture story lives at [synthesis.openie.dev](https://synthesis.openie.dev) — the strategic doctrine that locates pattern-lang in the seven-axis map of information synthesis.

## License

[Business Source License 1.1](./LICENSE). Licensed Work is the binaries attached to releases of this repository. Free for non-commercial use, internal use by organisations under $1M ARR, and security / academic research. Change Date is four years after each binary's release date; on that date the binary converts to Apache-2.0.

## Family

| | |
|---|---|
| [pattern-lang.ai](https://pattern-lang.ai) | This project — the public face |
| [openIE-dev/pattern-lang-core](https://github.com/openIE-dev/pattern-lang-core) | Source (private) |
| [openIE-dev/jouledb](https://github.com/openIE-dev/jouledb) | Energy-metered database |
| [openIE-dev/mgai](https://github.com/openIE-dev/mgai) | Math-Ground AI, sister project, same licensing pattern |
| [synthesis.openie.dev](https://synthesis.openie.dev) | The strategic doctrine |
| [joule-lang.dev](https://joule-lang.dev) | The energy-typed language |

## Contact

Issues and questions: open an issue here. The [security policy](./SECURITY.md) covers responsible disclosure.

For commercial-licensing inquiries (outside the BSL Additional Use Grant), see [pattern-lang.ai](https://pattern-lang.ai) or reach OpenIE via [openie.dev](https://openie.dev).

---

*Not affiliated with, or endorsed by, any AI vendor named in the documentation.*
