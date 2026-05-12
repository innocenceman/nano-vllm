# `gpu-execution/cache-runtime` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`kv-cache-runtime`](kv-cache-runtime/README.md) — `exceptioned-deep-partial` — Compute GPU KV-cache capacity from CUDA memory stats, allocate cache tensors, and attach per-layer key/value cache slices to attention modules.
