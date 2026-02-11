# Pattern

**Programs as Typed Pattern Graphs — Not Text Files That Encode Relationships**

[![License: CC BY-ND 4.0](https://img.shields.io/badge/License-CC%20BY--ND%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nd/4.0/)
[![Research](https://img.shields.io/badge/Research-Open%20Access-green.svg)](https://openie.dev)

> All code has already been written. Programming is retrieval and composition.

---

## What Is Pattern?

**Pattern** is a programming language where programs are typed directed acyclic graphs (DAGs) of computational patterns drawn from a universal catalog. There is no syntax. A program is a topology — a graph of typed nodes connected by typed edges.

- **Parallelism is topology.** Two branches with no shared edges are independent — discovered automatically, never declared.
- **Type safety is structural.** Edges connect typed ports. Incompatible types are rejected at construction.
- **One graph compiles to 37 languages.** Same topology, different code generation target.

```
Input(List<Int>)
    ├── find_max(List<Int>) → Int        ← parallel
    ├── sum(List<Int>) → Int             ← parallel
    └── sort(List<Int>) → List<Int>      ← parallel
            └── Output(List<Int>)
```

Three operations on the same data. No `async/await`. No thread pools. The graph shape *is* the execution schedule.

---

## By the Numbers

| Metric | Value |
|--------|-------|
| Base patterns | 800 across 28 domains |
| Target languages | 37 (Rust, C, Python, Go, COBOL, Haskell, ...) |
| Type-compatible compositions | 227,653 (from 413,449 possible pairs) |
| Average synthesis time | 23 microseconds per pattern |
| HumanEval | 164/164 (100%) |
| MBPP | 974/974 (100%) |
| Expanded catalog (Cartographer) | ~4,800 patterns across 48 domains |

---

## How It Works

### The Catalog Is the Language

In traditional languages, power comes from syntax — the ability to encode arbitrary computation. In Pattern, power comes from the **catalog** — the set of known computational patterns. Adding a pattern makes every program richer. Adding syntax makes every program more complex.

```
PatternSig {
    name: "merge_sort",
    inputs: [List<Int>],
    output: List<Int>,
    category: "sort",
    keywords: ["sort", "merge", "divide", "conquer", "stable"]
}
```

Two patterns compose when the output type of one is compatible with an input type of another. Composition is structural, not syntactic.

### Pattern Resolution Chain

When a program executes, each pattern node resolves through a layered lookup — first success wins:

| Layer | Mechanism | Latency | Description |
|-------|-----------|---------|-------------|
| L0 | Learned memory | 1 us | Cached prior results |
| Enlisted | Static catalog | 23 us | 800 base patterns |
| NCO | Domain recipes | ~100 us | 71 pre-validated compositions |
| L2 | Pure synthesis | ms | Deterministic — no LLM |
| L3 | Hybrid synthesis | 1-30s | LLM-guided, scaffold-augmented |

L0 absorbs every result. Each pattern is generated at most once. Every subsequent execution: microseconds.

### The Cartographer

The static 800 patterns cover algorithms and data structures. The **Cartographer** uses LLMs to enumerate the rest — operating systems, compilers, databases, network stacks — across 48 domains.

```
LLM (knows all patterns implicitly)
  → Cartographer (enumerate + type-sign + deduplicate)
    → DynamicCatalog (persistent runtime registry)
      → CodeFactory (lookup: Static → Dynamic → L3)
        → L0 absorbs result (microsecond retrieval forever)
```

The Cartographer discovers *signatures*, not implementations. Code is generated lazily on first use and cached permanently.

---

## Builder API

Programs are constructed through a fluent builder:

```rust
let catalog = build_pattern_signatures();
let mut b = ProgramBuilder::new("sort_and_dedup");

let input  = b.input(GraphType::List(Box::new(GraphType::Int)), "numbers");
let sort   = b.pattern("merge_sort", &catalog);
let dedup  = b.pattern("remove_duplicates", &catalog);
let output = b.output(GraphType::List(Box::new(GraphType::Int)));

b.connect(input, sort);
b.connect(sort, dedup);
b.connect(dedup, output);

let program = b.build();
// Topology: input → merge_sort → remove_duplicates → output
// Validation: 0 errors against static catalog
```

### Parallel Fork-Join — Discovered, Not Declared

```rust
let mut b = ProgramBuilder::new("parallel_stats");
let inp = b.input(GraphType::List(Box::new(GraphType::Int)), "data");
let mx  = b.pattern("find_max", &catalog);
let sm  = b.pattern("sum", &catalog);
let out = b.output(GraphType::Int);

b.connect(inp, mx);
b.connect(inp, sm);
// Execution plan: Stage 1 runs find_max and sum IN PARALLEL
// max_parallelism: 2 — from topology, not annotations
```

### Dynamic Patterns — Cartographer-Discovered

```rust
let mut b = ProgramBuilder::new("os_scheduler");
let procs   = b.input(GraphType::List(Box::new(GraphType::Int)), "processes");
let quantum = b.input(GraphType::Int, "time_quantum");
let sched   = b.pattern_typed(
    "dyn:round_robin_scheduler",
    vec![GraphType::List(Box::new(GraphType::Int)), GraphType::Int],
    GraphType::List(Box::new(GraphType::Int)),
);
let out = b.output(GraphType::List(Box::new(GraphType::Int)));

b.connect(procs, sched);
b.connect(quantum, sched);
b.connect(sched, out);
// Resolution: L3 generates code on first execution; L0 caches permanently.
// The pattern was discovered by the Cartographer, not hand-coded.
```

---

## Type System

8 base types with structural compatibility:

| Type | Description |
|------|-------------|
| `Int` | 64-bit signed integer |
| `Float` | IEEE 754 double-precision |
| `Bool` | Boolean value |
| `String` | UTF-8 text |
| `List<T>` | Homogeneous ordered collection |
| `Tuple<T1,...,Tn>` | Fixed-width heterogeneous product |
| `Option<T>` | Nullable/optional wrapper |
| `Any` | Polymorphic wildcard — compatible with everything |

`List<Int>` connects to `List<Any>`. `Tuple<Int, Float>` connects to `Tuple<Int, Float>`. Types are checked structurally, not nominally.

---

## Style Engine — Same Algorithm, Different Soul

Pattern separates correctness from personality. A continuous **4D style space** transforms correct code into code that reads like Kernighan, Knuth, Torvalds, or Dijkstra wrote it.

| Axis | Range | Controls |
|------|-------|----------|
| `voice` | 0.0 - 1.0 | Comment density (silent → poetic) |
| `doc_level` | 0.0 - 1.0 | Documentation depth (none → academic) |
| `naming` | 0.0 - 1.0 | Identifier verbosity (single-char → mathematical) |
| `header` | 0.0 - 1.0 | Header weight (none → full attribution) |

**8 Named Presets:**

| Preset | voice | doc | naming | header |
|--------|-------|-----|--------|--------|
| **kernighan** | 0.25 | 0.33 | 0.00 | 0.33 |
| **knuth** | 0.75 | 1.00 | 0.66 | 1.00 |
| **torvalds** | 0.25 | 0.00 | 0.33 | 0.33 |
| **dijkstra** | 0.50 | 0.66 | 1.00 | 0.66 |
| **pythonista** | 0.50 | 0.66 | 0.66 | 0.66 |
| **rustacean** | 0.50 | 0.33 | 0.66 | 0.33 |
| **hacker** | 0.25 | 0.00 | 0.00 | 0.00 |
| **clean** | 0.50 | 0.66 | 0.33 | 0.66 |

Custom coordinates. Linear interpolation between presets. GPU-accelerated style search (160K candidates in ~10ms via Apple Metal).

---

## Supported Languages (37)

| Category | Languages |
|----------|-----------|
| **Systems** | Rust, C, C++, Zig, D, V, Ada |
| **JVM/CLR** | Java, C#, Kotlin, Scala, Groovy |
| **Legacy** | COBOL, Fortran |
| **Scripting** | Python, JavaScript, TypeScript, Ruby, PHP, Perl, Lua, Bash |
| **Functional** | Haskell, OCaml, F#, Elixir, Clojure |
| **Modern** | Go, Swift, Dart, Julia, R, Nim, Mojo |
| **Emerging** | AssemblyScript, Grain, MoonBit |

Same graph topology. Different code generation target. 23 microseconds average.

---

## Pattern Domains (28 + 20 Cartographer)

**Static catalog (28):** Math, Graph, DP, String, Tree, Search, Sort, ML, Distributed, Database, Networking, Crypto, Imaging, Signal, Control, Quantum, Physics, Bioinformatics, Verification, Finance, Optimization, Compression, Error Correction, Graphics, Robotics, IR, Audio, GIS

**Cartographer expansion (+20):** OS Kernel, Filesystem, Device Driver, Window System, Shell, Compiler Frontend, Compiler Backend, Virtual Machine, Container, Package Manager, Build System, Text Editor, Version Control, Web Server, Web Client, Testing, Logging, Monitoring, Serialization, Concurrency

---

## Architecture

```
Problem (natural language or keyword query)
    │
    ▼
OODA Engagement Loop
    │
    ├─ Mount ────── L0 Learned Memory (1 us, cached)
    ├─ SideControl ─ Enlisted (800 patterns, 23 us)
    ├─ Doctrine ──── NCO (71 domain recipes, ~100 us)
    ├─ Guard ─────── Generic 2-pattern composition
    ├─ HalfGuard ─── Task decomposition
    ├─ Scramble ──── Adaptive retry
    └─ Tap ──────── L2 Pure Synthesis → L3 LLM Hybrid
    │
    ▼
Style Engine (4D post-processing)
    │
    ▼
Cortex IR → CodeGen → 37 Target Languages
```

---

## Research Papers

| # | Paper | PDF |
|---|-------|-----|
| 08 | [The Pattern Programming Language](docs/PATTERN_LANGUAGE_SPECIFICATION.md) | [PDF](docs/08-Pattern-Programming-Language.pdf) |
| 09 | [The Cartographer: LLM-Driven Catalog Expansion](docs/CARTOGRAPHER_WHITEPAPER.md) | [PDF](docs/09-Cartographer-Catalog-Expansion.pdf) |
| 10 | [The Style Engine: 4D Code Personality](docs/STYLE_ENGINE_WHITEPAPER.md) | [PDF](docs/10-Style-Engine-4D-Code-Personality.pdf) |

---

## License

This work is licensed under [CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0/) (Creative Commons Attribution-NoDerivatives 4.0 International).

You are free to use and share this work for any purpose, including commercial use, with attribution. You may not distribute modified versions.

Copyright (c) 2025-2026 Open Interface Engineering, Inc. (openIE)

---

**David Jean Charlot, PhD** | Open Interface Engineering, Inc. (openIE) | University of California, Santa Barbara (UCSB)

[david@openie.dev](mailto:david@openie.dev) | [dcharlot@ucsb.edu](mailto:dcharlot@ucsb.edu)
