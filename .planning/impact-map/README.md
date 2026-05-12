# Brownfield Impact Map

**Generated:** 2026-05-12  
**Mode:** Verified + GSD bridge  
**Scope:** Whole repository  

This impact map is a hierarchical planning handoff: architecture domain → subsystem/group → final leaf. Final leaf directories contain `README.md`, `code-paths.md`, `file-roles.md`, and `change-to-test.md`.

## Evidence Used

- .planning/codebase/*.md consumed; 7 docs, 1195 total lines from the prior map pass.
- npx gitnexus analyze refreshed index at commit caa4415; status up-to-date; 424 nodes, 734 edges, 17 clusters, 17 flows; embeddings preserved.
- code-review-graph update completed; status at commit caa4415; 158 nodes, 742 edges, 21 files; detect-changes reported 7 docs-only files, 0 changed functions/classes, risk 0.00.
- graphify and gsd graphify query commands checked; unavailable: no graphify binary and gsd-sdk query graphify/graphify.status returned unknown command.
- python3 -m compileall -q nanovllm example.py bench.py passed.
- Dependency probe failed for torch, triton, transformers, flash_attn, xxhash, numpy, safetensors, tqdm; package import smoke failed before runtime validation.

## Domain Overview

| Domain | Groups | Leaves | Primary owner/risk |
| --- | ---: | ---: | --- |
| `public-api` | 2 | 3 | Import/API compatibility and config validation. |
| `engine-runtime` | 3 | 4 | CPU-side scheduling, state, and prefix-cache policy. |
| `gpu-execution` | 4 | 4 | CUDA/NCCL/process-control and graph execution. |
| `model-and-weights` | 2 | 2 | Qwen3-only model composition and safetensors mapping. |
| `layer-primitives` | 4 | 5 | Distributed tensor/attention/math kernels. |
| `repository-ops` | 2 | 2 | Manual scripts, dependency metadata, and docs. |


## Coverage Summary

| Label | Count |
| --- | ---: |
| `verified` | 1 |
| `deep-partial` | 2 |
| `exceptioned-deep-partial` | 17 |

No placeholder labels remain. No final leaf is a coarse top-level directory mirror; each leaf has one primary responsibility, bounded source paths, an impact anchor, risk surface, and validation ladder.
