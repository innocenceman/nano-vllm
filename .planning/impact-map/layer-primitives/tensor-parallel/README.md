# `layer-primitives/tensor-parallel` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`tensor-parallel-linears`](tensor-parallel-linears/README.md) — `exceptioned-deep-partial` — Provide replicated, column-parallel, merged-column, QKV, and row-parallel linear layers with sharded weight loaders and distributed reductions.
- [`embedding-lm-head`](embedding-lm-head/README.md) — `exceptioned-deep-partial` — Shard vocab embeddings and gather LM-head logits across tensor-parallel ranks while selecting final prefill logits via context.
