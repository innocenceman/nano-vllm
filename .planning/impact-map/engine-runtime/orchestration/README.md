# `engine-runtime/orchestration` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`request-lifecycle`](request-lifecycle/README.md) — `exceptioned-deep-partial` — Turn prompts and `SamplingParams` into scheduled `Sequence` work, call the model runner, decode finished outputs, and manage shutdown.
