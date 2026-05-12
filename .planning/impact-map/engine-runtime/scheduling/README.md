# `engine-runtime/scheduling` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`scheduler-policy`](scheduler-policy/README.md) — `deep-partial` — Admit waiting sequences, choose prefill versus decode batches, enforce token/sequence budgets, preempt cache-starved decode work, and postprocess completions.
- [`kv-block-manager`](kv-block-manager/README.md) — `exceptioned-deep-partial` — Allocate/deallocate KV block IDs, maintain reference counts, hash full prompt blocks, and expose prefix-cache reuse decisions to scheduler and runner.
