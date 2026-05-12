# `gpu-execution/process-control` Group

Generated: 2026-05-12

This is an intermediate architecture grouping. Final leaf packets are below; this grouping file intentionally avoids pretending the group itself is a testable leaf.

## Leaves

- [`distributed-process-control`](distributed-process-control/README.md) — `exceptioned-deep-partial` — Initialize NCCL process groups, select CUDA rank, coordinate tensor-parallel child processes through shared memory, and tear down distributed state.
