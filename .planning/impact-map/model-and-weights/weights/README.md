# `model-and-weights/weights` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`weight-loading`](weight-loading/README.md) — `exceptioned-deep-partial` — Load Hugging Face safetensors files, apply packed Q/K/V and gate/up mappings, and dispatch to parameter-specific sharding loaders.
