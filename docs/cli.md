# CLI reference

`pattern-lang <subcommand>` — the lawful synthesizer and cascade dispatcher in one binary.

## Subcommands

| Command | What it does |
|---|---|
| `engage` | Interactive query/response with doctrine config. The default mode for exploring the catalog. |
| `compose <intent>` | Map an intent to a typed DAG of patterns. |
| `list [--category <name>]` | Show patterns by category or all of them. |
| `vocab` | Print vocabulary statistics (counts by category, dynamic vs static). |
| `run <pattern> <args...>` | Execute a single pattern directly (e.g. `run gcd 12 8`). |
| `solve <problem>` | Dispatch to a constraint-satisfaction solver (SAT, Sudoku, N-Queens, graph coloring). |
| `emit --target <lang>` | Emit the most recent composition in any of 41 target languages. |
| `bench [cascade]` | Run the in-tree benchmark suite. `bench cascade` reports the verifier-first cost wedge. |
| `repl` | Read-eval-print loop. |

## Global flags

- `--target <lang>` — emission target (default: `rust`)
- `--style <preset>` — Kernighan / Knuth / Torvalds / Dijkstra / Pythonista / Rustacean / Hacker / Clean
- `--budget-joules <n>` — refuse to dispatch beyond this energy budget
- `--explain` — include the cascade walk and verifier verdicts in the output
- `--json` — machine-readable output

## Examples

```bash
# Single pattern
pattern-lang run binary_search "1,3,5,7,9,11" 7

# Compose + emit
pattern-lang compose "tokenize a string and count word frequencies"
pattern-lang emit --target python --style pythonista

# Constraint solve
pattern-lang solve sat "(a OR b) AND (NOT a OR c)"

# Bench with explain
pattern-lang bench cascade --explain --json
```
