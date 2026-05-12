# `public-api/configuration` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`config-normalization`](config-normalization/README.md) — `exceptioned-deep-partial` — Normalize engine capacity, tensor-parallel, eager/graph, KV-cache, and Hugging Face model metadata.
- [`sampling-parameters`](sampling-parameters/README.md) — `verified` — Carry per-request sampling knobs from public API into sequence state and sampler temperature tensors.
