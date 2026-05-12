# `layer-primitives/attention` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`attention-kernels`](attention-kernels/README.md) — `exceptioned-deep-partial` — Write keys/values into KV cache with Triton and invoke FlashAttention varlen/paged-cache kernels for prefill and decode.
