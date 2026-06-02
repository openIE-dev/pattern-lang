# Getting started

Pattern-lang ships as release binaries. No build step on your machine.

## 1. Install

Once a release is published, the shortest path is:

```bash
curl -fsSL https://github.com/openIE-dev/pattern-lang/releases/latest/download/install.sh | sh
pattern-lang --version
```

For air-gapped installs, grab the tarball for your platform from the [Releases](https://github.com/openIE-dev/pattern-lang/releases) page and place the binary on `$PATH`.

## 2. First query

```bash
pattern-lang engage
> compose: greatest common divisor
> target: rust
```

You get a typed DAG, an emitted Rust function, and a receipt that names the cascade tier and the joules spent.

## 3. Run as a server

```bash
joule-edge-server --bind 127.0.0.1:7777
```

The server exposes an Anthropic-compatible `POST /v1/messages` endpoint. Every response includes `usage.joules_spent` and `usage.tier_used`.

```bash
curl http://127.0.0.1:7777/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "pattern-lang",
    "messages": [{"role": "user", "content": "factorial of 7"}]
  }'
```

## 4. Where the cascade went

```bash
pattern-lang bench cascade
```

Prints the in-tree cost wedge: `verify_first` vs `always-frontier` on a 60-query workload, with per-tier joule breakdown.

## Next

- [CLI reference](./cli.md)
- [Server reference](./server.md)
- [Cascade and verifiers](./cascade.md)
- [License FAQ](./license-faq.md)
