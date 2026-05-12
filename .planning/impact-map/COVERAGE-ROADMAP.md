# Coverage and Evidence Roadmap

**Generated:** 2026-05-12

## Readiness Verdict

The repository is source-mapped and graph-indexed, but most leaves are not behavior-verified because the current environment lacks runtime dependencies (`torch`, `triton`, `transformers`, `flash_attn`, `xxhash`, `numpy`, `safetensors`, `tqdm`), no automated tests exist, and GPU/model fixtures are unavailable. The handoff is ready for GSD planning, not ready to claim runtime correctness.

## Immediate Coverage Priorities

1. **Packaging/import contract:** add `tqdm` to dependency metadata or remove eager import dependency; add a clean-env public API import smoke.
2. **CPU unit-test harness:** add pytest or equivalent for `SamplingParams`, `Sequence`, `Scheduler`, and `BlockManager` using pure/fake dependencies where possible.
3. **Dependency fixture layer:** define optional GPU markers and tiny Hugging Face config/model fixtures for Qwen3/loader/model-runner tests.
4. **ModelRunner seam split:** extract process-control, batch-prep, KV-cache allocation, and CUDA graph behavior into separately testable seams before feature work.
5. **CI/check contract:** run compile/import/unit checks in CI; keep GPU tests marked/optional.

## Tool Readiness Gate Results

| Tool/check | Readiness | Evidence |
| --- | --- | --- |
| GitNexus | available and refreshed | npx gitnexus analyze refreshed index at commit caa4415; status up-to-date; 424 nodes, 734 edges, 17 clusters, 17 flows; embeddings preserved. |
| code-review-graph | available and updated | code-review-graph update completed; status at commit caa4415; 158 nodes, 742 edges, 21 files; detect-changes reported 7 docs-only files, 0 changed functions/classes, risk 0.00. |
| graphify | unavailable | graphify and gsd graphify query commands checked; unavailable: no graphify binary and gsd-sdk query graphify/graphify.status returned unknown command. |
| Python compile | passed | python3 -m compileall -q nanovllm example.py bench.py passed. |
| Runtime import | blocked | Dependency probe failed for torch, triton, transformers, flash_attn, xxhash, numpy, safetensors, tqdm; package import smoke failed before runtime validation. |

## Label Counts

| Label | Count | Promotion path |
| --- | ---: | --- |
| `verified` | 1 | Keep passing source/check evidence; add behavioral tests for future changes. |
| `deep-partial` | 2 | Add targeted tests to promote. |
| `exceptioned-deep-partial` | 17 | Resolve missing dependency/GPU/model/test blocker first. |

## Per-Leaf Next Validation

### `public-api/api-surface/package-facade` — `exceptioned-deep-partial`

- After installing project dependencies, run `python3 -c "from nanovllm import LLM, SamplingParams"`.
- Add an import-only test that catches missing metadata dependencies such as `tqdm`.

### `public-api/configuration/config-normalization` — `exceptioned-deep-partial`

- Unit-test invalid model path, invalid tensor-parallel size, invalid KV block size.
- Use a tiny local Hugging Face config fixture to avoid downloading model weights.

### `public-api/configuration/sampling-parameters` — `verified`

- Add CPU unit tests for temperature guard, max token validation, and copying values into `Sequence`.

### `engine-runtime/orchestration/request-lifecycle` — `exceptioned-deep-partial`

- Import-only test for public package.
- CPU fake-runner test for `generate` loop ordering and output decode.
- Lifecycle test that `exit` is idempotent.

### `engine-runtime/state/sequence-state` — `deep-partial`

- CPU unit tests for empty prompt, block slicing, completion slicing, append, and pickle round trip.

### `engine-runtime/scheduling/scheduler-policy` — `deep-partial`

- CPU unit tests with fake `BlockManager` capacity.
- Regression tests for chunked prefill and preemption ordering.

### `engine-runtime/scheduling/kv-block-manager` — `exceptioned-deep-partial`

- CPU unit tests for allocation reuse, deallocation order, prefix-cache hits/misses, collision guard, and append capacity.

### `gpu-execution/process-control/distributed-process-control` — `exceptioned-deep-partial`

- Add injectable rendezvous/shared-memory names.
- Add fake child-process RPC tests for payload size and exit ordering.

### `gpu-execution/batching/batch-preparation-context` — `exceptioned-deep-partial`

- Extract CPU-pure slot-mapping computation for unit tests.
- Add context reset test around runner failures.

### `gpu-execution/cache-runtime/kv-cache-runtime` — `exceptioned-deep-partial`

- GPU-marked integration test for cache allocation with tiny config.
- Unit-test layer-id/cache attachment with fake modules.

### `gpu-execution/execution-mode/cuda-graph-execution` — `exceptioned-deep-partial`

- GPU integration test comparing eager and graph decode for deterministic logits.
- Exit test for eager/non-eager cleanup.

### `model-and-weights/model-definition/qwen3-composition` — `exceptioned-deep-partial`

- Instantiate with tiny Qwen3 config under single-process distributed fixture.
- Test tied embeddings and packed mapping coverage.

### `model-and-weights/weights/weight-loading` — `exceptioned-deep-partial`

- Fixture safetensors file with minimal model parameters.
- Unit-test packed mapping and loader dispatch with fake parameters.

### `layer-primitives/tensor-parallel/tensor-parallel-linears` — `exceptioned-deep-partial`

- Fake distributed rank/world-size tests for shard offsets.
- CPU tests for `divide` and weight_loader slicing.

### `layer-primitives/tensor-parallel/embedding-lm-head` — `exceptioned-deep-partial`

- Distributed fixture tests for vocab partition masking and logits gather.
- Context-driven prefill last-index test.

### `layer-primitives/attention/attention-kernels` — `exceptioned-deep-partial`

- GPU tests for store_kvcache slot mapping including -1 slots.
- Compare prefill/decode attention outputs against a reference for tiny tensors.

### `layer-primitives/math/math-primitives` — `exceptioned-deep-partial`

- CPU/GPU numeric tests against simple reference implementations.
- Cache identity test for `get_rope` LRU behavior.

### `layer-primitives/sampling/sampling-kernel` — `exceptioned-deep-partial`

- Seeded statistical smoke test for sampler.
- Validation tests for temperature propagation from params to CUDA tensor.

### `repository-ops/scripts/manual-scripts` — `exceptioned-deep-partial`

- CLI smoke test that skips when model/GPU unavailable.
- Benchmark config arguments instead of hard-coded path.

### `repository-ops/packaging/packaging-docs` — `exceptioned-deep-partial`

- Add package metadata test that imports public API in a clean install.
- Add README smoke command guarded by model availability.

