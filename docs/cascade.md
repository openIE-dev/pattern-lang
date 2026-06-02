# Cascade and verifiers

The cascade is a tier registry with cost-ordered routing and verifier-gated acceptance.

## Tiers

| Tier | Cost class | Examples | When it answers |
|---|---|---|---|
| **L0 cache** | ~nJ | scar memory, episode memory | Same (intent, target) pair the system has resolved before |
| **L1 lawful** | ~µJ | the 690-primitive catalog, Cortex IR | Intent matches a typed pattern composition |
| **L2 small** | ~µJ–mJ | Prism (MLGRU on ternary weights), Liquid (CfC cells), MRL (Matryoshka retrieval), DeBERTa (NLI) | Pattern-bound work with a retrieval anchor |
| **L3 medium** | ~mJ | Gemma 4 (Q8 / Q5 / Q4_K), LMM (vision+text) | Open-ended generation, multimodal input |
| **L4 frontier** | ~10s mJ | RPC out to a frontier model | Only when every cheaper tier defers |

Coordinates on the seven-axis Synthesis lattice (Z, E, T, P, I, V, R) are declared per tier — see [synthesis.openie.dev](https://synthesis.openie.dev) for the map.

## Routing policies

Set with `--verify` on the server, or per-request in JSON:

- **`verify_first`** (default) — walk tiers in cost order, accept the first verifier-approved answer. Real benchmark: 2.86× cheaper than always-frontier at 100% correctness.
- **`verify_all`** — run every applicable tier, sample from a Boltzmann posterior over (confidence, cost). Use for calibration trace collection or when you want stochastic exploration.
- **`none`** — accept the first tier's answer without verification. Only useful for benchmarking against an unverified baseline.

## Verifier types

Each tier may be wrapped in any of:

- **Recompute oracle** — run the lawful primitive on the same input and compare. Zero hallucination by construction.
- **EBM constraint** — wrap a constraint-satisfaction problem (SAT, Sudoku, N-Queens, graph coloring) as a `Verifier`. Zero energy means satisfied.
- **Lean proof** — emit Lean 4 theorem syntax for deterministic arithmetic claims (`Nat.gcd`, `Nat.factorial`, `Nat.Prime`). Optional kernel certification via the actual Lean compiler.
- **K-of-N voting** — ensemble n heterogeneous verifiers; accept when k agree. Confidence on accept = min over accepting verifiers (weakest link binds).

## The asymmetry

Generation and verification are asymmetric — a model that generates IMO solutions at ~15% accuracy verifies them at ~85%. Pattern-lang makes that mechanical: wrap a small generator in an independent verifier, escalate only when the verifier rejects.

## Cost contract

A verifier's `estimated_joules` must be strictly smaller than the average cost of the tiers it gates. The library does not enforce this. A verifier more expensive than its tier is a net loss.

## Inspecting a walk

```bash
# After a request, the cascade trace is stored under the request id
curl http://127.0.0.1:7777/cascade/explain?id=req_…
```

Returns the tier sequence, per-tier joule estimate vs measured, verifier verdicts, and the final accepted answer's provenance.
