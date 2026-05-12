# Brownfield Handoff to GSD Mainline

**Generated:** 2026-05-12  
**Mode:** Verified + GSD bridge  
**Scope:** Whole repository  
**Repository:** `/home/yanx/projects/vllm/nano-vllm`  
**Baseline commit:** `caa4415`

This packet recommends next GSD work. It does **not** authorize or perform GSD mainline mutations. The following files were intentionally left untouched by this skill: `.planning/PROJECT.md`, `.planning/REQUIREMENTS.md`, `.planning/ROADMAP.md`, `.planning/STATE.md`, and phase directories.

## Readiness Verdict

**Verdict:** Ready for `$gsd-new-project` with brownfield evidence, but not ready to claim runtime correctness.

The repository has a strong source/architecture map and refreshed graph evidence. Most runtime leaves remain `exceptioned-deep-partial` because the current environment lacks project runtime dependencies and no automated tests exist. The first GSD milestone should stabilize import/package/test harness before major model-runner or GPU feature work.

## Artifacts Written

- `.planning/architecture/ARCHITECTURE-ATLAS.md`
- `.planning/impact-map/README.md`
- `.planning/impact-map/MODULE-INDEX.md`
- `.planning/impact-map/COVERAGE-ROADMAP.md`
- `.planning/impact-map/**/README.md`
- `.planning/impact-map/**/leaf-index.md`
- `.planning/impact-map/**/{README.md,code-paths.md,file-roles.md,change-to-test.md}` for every final leaf
- `.planning/handoff/BROWNFIELD-HANDOFF.md`

## Module Counts

| Metric | Count |
| --- | ---: |
| Domains | 6 |
| Groups | 17 |
| Final leaves | 20 |
| `verified` leaves | 1 |
| `deep-partial` leaves | 2 |
| `exceptioned-deep-partial` leaves | 17 |

## Validation Evidence Used

| Evidence | Result |
| --- | --- |
| Codebase map | .planning/codebase/*.md consumed; 7 docs, 1195 total lines from the prior map pass. |
| GitNexus | npx gitnexus analyze refreshed index at commit caa4415; status up-to-date; 424 nodes, 734 edges, 17 clusters, 17 flows; embeddings preserved. |
| code-review-graph | code-review-graph update completed; status at commit caa4415; 158 nodes, 742 edges, 21 files; detect-changes reported 7 docs-only files, 0 changed functions/classes, risk 0.00. |
| graphify | graphify and gsd graphify query commands checked; unavailable: no graphify binary and gsd-sdk query graphify/graphify.status returned unknown command. |
| Compile check | python3 -m compileall -q nanovllm example.py bench.py passed. |
| Runtime dependency/import probe | Dependency probe failed for torch, triton, transformers, flash_attn, xxhash, numpy, safetensors, tqdm; package import smoke failed before runtime validation. |
| Source inventory | Source inventory counted 21 Python files; no open task markers; asserts present in config, engine, layers, model, and sampling modules. |

## Top Risks

1. **Import/package contract is broken in this environment:** runtime imports fail because dependencies are not installed; `tqdm` is imported but missing from `pyproject.toml` dependencies.
2. **No automated tests or CI:** scheduler, block manager, model runner, CUDA graph, loader, and layers have no regression guards.
3. **`ModelRunner` is a broad blast-radius file:** it owns distributed setup, shared-memory RPC, model loading, warmup, KV-cache allocation, batch preparation, sampling, and CUDA graph replay.
4. **Hard-coded distributed coordination:** NCCL rendezvous and shared memory use fixed names/ports.
5. **Assert-heavy validation:** public/user-facing invalid states often fail through `assert` or implicit exceptions.
6. **GPU/model fixture gap:** runtime correctness cannot be verified without installed dependencies, local model fixtures, and GPU/NCCL availability.

## Suggested Backlog Candidates

1. **Package/import smoke and dependency metadata** — add `tqdm` dependency or remove eager import dependency; add clean import smoke. Affects `repository-ops/packaging-docs`, `public-api/package-facade`, `engine-runtime/request-lifecycle`.
2. **CPU test harness for pure engine state** — introduce tests for `SamplingParams`, `Sequence`, `Scheduler`, and `BlockManager` before behavior changes. Affects `public-api/sampling-parameters`, `engine-runtime/sequence-state`, `engine-runtime/scheduler-policy`, `engine-runtime/kv-block-manager`.
3. **Typed validation pass** — replace public-facing asserts with `ValueError`/`RuntimeError` and clear messages. Affects config, sampling, sequence, block manager, layers, and model composition.
4. **ModelRunner seam hardening** — split or isolate rendezvous/shared-memory configuration, batch prep, cache allocation, and CUDA graph capture behind testable seams.
5. **Tiny model/GPU validation lane** — define optional GPU-marked tests and a tiny local Hugging Face config/model fixture for Qwen3/loader/model-runner paths.

## Recommended Next GSD Command

```text
$gsd-new-project
```

Use the existing `.planning/codebase/**`, `.planning/architecture/ARCHITECTURE-ATLAS.md`, `.planning/impact-map/**`, and this handoff as brownfield inputs. If external planning documents already exist elsewhere, use `$gsd-ingest-docs` instead.

## Mainline Mutation Boundary

This handoff did not create or mutate:

- `.planning/PROJECT.md`
- `.planning/REQUIREMENTS.md`
- `.planning/ROADMAP.md`
- `.planning/STATE.md`
- `.planning/phases/**`
- backlog files

GSD mainline should adopt recommendations through its own workflow.
