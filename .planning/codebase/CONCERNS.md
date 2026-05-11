# Codebase Concerns

**Analysis Date:** 2026-05-11

## Tech Debt

**Validation via `assert` and implicit crashes:**
- Issue: Runtime input/config validation uses assertions and indexing instead of typed exceptions; assertions can be stripped with `python -O`, and user errors surface as `AssertionError` or `IndexError`.
- Files: `nanovllm/config.py`, `nanovllm/sampling_params.py`, `nanovllm/engine/sequence.py`, `nanovllm/layers/linear.py`, `nanovllm/models/qwen3.py`
- Impact: Invalid model paths, unsupported tensor parallel sizes, zero/negative sampling limits, empty prompts, and unsupported model shapes fail late or opaquely.
- Fix approach: Replace public-facing asserts with `ValueError`/`RuntimeError` messages and validate `SamplingParams.max_tokens`, prompt length, prompt emptiness, model path, dtype/head divisibility before scheduling.

**`LLM` public API is a pass-through subclass:**
- Issue: Public class is only `class LLM(LLMEngine): pass`, so API semantics live in engine internals.
- Files: `nanovllm/llm.py`, `nanovllm/engine/llm_engine.py`, `nanovllm/__init__.py`
- Impact: API compatibility shims, lifecycle docs, and user-facing validation must couple directly to scheduler/model-runner internals.
- Fix approach: Keep `LLM` as the public facade with documented constructor/generate/close behavior and delegate implementation to a private engine object.

**No repository test, lint, or CI contract:**
- Issue: No test files, pytest config, lint config, type-check config, or CI pipeline detected.
- Files: `pyproject.toml`, `nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`
- Impact: Scheduler, KV cache, tensor-parallel, CUDA graph, and loader changes have no automated regression guard; breakage appears only during manual GPU runs.
- Fix approach: Add CPU-safe unit tests for pure scheduler/block logic, GPU-marked integration tests for model runner/attention, and CI jobs that run formatting, import/AST/compile checks, and non-GPU unit tests.

**Hard-coded single-node runtime coordination:**
- Issue: Distributed setup uses `tcp://localhost:2333` and shared memory name `"nanovllm"` with a fixed 1 MiB buffer.
- Files: `nanovllm/engine/model_runner.py`
- Impact: Concurrent engines on the same host can collide; large control payloads can overflow shared memory; port conflicts produce opaque NCCL initialization failures.
- Fix approach: Generate per-engine rendezvous ports and shared-memory names, validate serialized payload size before writing, and expose process-group configuration through `Config`.

**Model-specific implementation is embedded as the only architecture path:**
- Issue: Qwen3 is the only model implementation and loader mapping target.
- Files: `nanovllm/models/qwen3.py`, `nanovllm/engine/model_runner.py`, `nanovllm/utils/loader.py`
- Impact: Adding another architecture requires editing `ModelRunner` and weight mapping code rather than registering a model family.
- Fix approach: Add a model registry keyed by `hf_config.model_type` and keep architecture-specific packed weight mappings inside model modules.

## Known Bugs

**`generate` silently drops requests when sampling parameter list length differs:**
- Symptoms: `zip(prompts, sampling_params)` enqueues only matched pairs while progress total remains `len(prompts)`.
- Files: `nanovllm/engine/llm_engine.py`
- Trigger: Call `LLM.generate(["a", "b"], [SamplingParams()])`.
- Workaround: Pass one `SamplingParams` object or a list with exactly one item per prompt.

**`generate` return type annotation disagrees with returned objects:**
- Symptoms: Signature says `-> list[str]`, implementation returns dictionaries with `text` and `token_ids`, matching the README example.
- Files: `nanovllm/engine/llm_engine.py`, `README.md`, `example.py`
- Trigger: Static typing, generated docs, or callers expecting `list[str]`.
- Workaround: Treat outputs as dictionaries with `text` and `token_ids`.

**Zero or negative `max_tokens` does not terminate correctly:**
- Symptoms: `SamplingParams` accepts `max_tokens <= 0`; scheduler only finishes after appending a token and checking equality, so `0` overshoots and negative values never match.
- Files: `nanovllm/sampling_params.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/sequence.py`
- Trigger: `SamplingParams(max_tokens=0)` or `SamplingParams(max_tokens=-1)`, especially with `ignore_eos=True`.
- Workaround: Only pass positive `max_tokens`.

**Empty prompt token lists crash sequence construction:**
- Symptoms: `Sequence.__init__` reads `token_ids[-1]` without validating non-empty inputs.
- Files: `nanovllm/engine/sequence.py`, `nanovllm/engine/llm_engine.py`
- Trigger: `LLM.generate([[]], SamplingParams())` or tokenizer output that encodes to an empty token list.
- Workaround: Ensure every prompt has at least one token before calling `generate`.

**Multiple local tensor-parallel engines collide on fixed rendezvous resources:**
- Symptoms: A second tensor-parallel engine can fail to initialize or attach to the wrong shared-memory name/port.
- Files: `nanovllm/engine/model_runner.py`
- Trigger: Start two `LLM(..., tensor_parallel_size > 1)` instances on one host.
- Workaround: Run one tensor-parallel engine per host/session.

## Security Considerations

**Pickle-based shared-memory RPC:**
- Risk: Worker ranks call `pickle.loads` on bytes in a predictable shared-memory segment and then dispatch method names dynamically.
- Files: `nanovllm/engine/model_runner.py`
- Current mitigation: Shared-memory RPC is intended for local worker processes spawned by the same engine, not remote clients.
- Recommendations: Replace pickle with a constrained schema (method enum plus tensor metadata), randomize shared-memory names, check payload sizes, and reject unknown methods before dispatch.

**Dynamic method dispatch allows internal RPC broadening:**
- Risk: `ModelRunner.call` accepts any method name and uses `getattr` directly; compromised or corrupted control data can invoke unintended methods.
- Files: `nanovllm/engine/model_runner.py`
- Current mitigation: Rank 0 is the only writer in normal operation.
- Recommendations: Use an explicit allowlist for `run` and `exit`, validate argument shapes, and fail closed on unknown method names.

**Local model loading has no manifest validation:**
- Risk: Loader trusts every `*.safetensors` file under the supplied model path and does not report missing, unexpected, duplicate, or shape-incompatible weights before copy attempts.
- Files: `nanovllm/utils/loader.py`, `nanovllm/config.py`
- Current mitigation: `safetensors` avoids Python pickle model files; `AutoConfig.from_pretrained` reads the local model directory.
- Recommendations: Check an expected weight manifest from config/model architecture, validate shapes before copy, and report a complete load summary.

**Environment secret handling:**
- Risk: No env files detected; `.gitignore` excludes `.env` and `.pypirc`. Runtime examples use local model paths rather than credentials.
- Files: `.gitignore`, `README.md`, `example.py`, `bench.py`
- Current mitigation: No secret-bearing files found during scan.
- Recommendations: Keep credentials out of examples and document Hugging Face authentication through standard external CLI/session configuration, not committed files.

## Performance Bottlenecks

**Per-step CPU list assembly and tensor allocation:**
- Problem: Prefill/decode paths rebuild Python lists and new pinned CPU tensors every scheduler step, then copy them to CUDA.
- Files: `nanovllm/engine/model_runner.py`
- Cause: `prepare_prefill`, `prepare_decode`, `prepare_sample`, and `prepare_block_tables` allocate fresh tensors for input ids, positions, slot mappings, context lengths, and temperatures.
- Improvement path: Reuse staging buffers sized to configured maxima, batch-copy into preallocated pinned tensors, and add microbenchmarks around preparation time versus model time.

**Prefix-cache hashing uses NumPy conversion per block:**
- Problem: Hashing converts token lists to NumPy arrays for each block and recomputes prefix hashes on scheduling/hash finalization.
- Files: `nanovllm/engine/block_manager.py`
- Cause: `compute_hash` allocates `np.array(token_ids)` and serializes it for every block.
- Improvement path: Store block hashes on sequences, use explicit dtype serialization, and benchmark incremental token hashing.

**Full-vocabulary sampling has no filtering controls:**
- Problem: Sampling materializes full softmax probabilities and Gumbel noise over the entire vocabulary for every decode step.
- Files: `nanovllm/layers/sampler.py`, `nanovllm/engine/model_runner.py`
- Cause: `Sampler.forward` computes `torch.softmax(logits, dim=-1)` and `torch.empty_like(probs).exponential_`.
- Improvement path: Add top-k/top-p/min-p controls and optimized kernels or vectorized filtering paths for large vocabularies.

**CUDA graph capture scales memory with batch-size ladder:**
- Problem: Startup captures many CUDA graphs up to `min(max_num_seqs, 512)`.
- Files: `nanovllm/engine/model_runner.py`
- Cause: `capture_cudagraph` builds graphs for `[1, 2, 4, 8] + range(16, max_bs + 1, 16)`.
- Improvement path: Make graph batch sizes configurable and collect memory/time metrics for graph capture versus eager decode.

**Tokenizer and output decoding are synchronous in `generate`:**
- Problem: Prompt encoding and final decoding happen inline on the caller thread.
- Files: `nanovllm/engine/llm_engine.py`, `example.py`, `bench.py`
- Cause: `add_request` encodes strings one by one; `generate` decodes all completions after scheduling finishes.
- Improvement path: Support pre-tokenized input as the primary high-throughput path, validate list lengths, and optionally stream decoded outputs incrementally.

## Fragile Areas

**Scheduler + block manager invariants:**
- Files: `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`, `nanovllm/engine/sequence.py`
- Why fragile: Correctness depends on `num_cached_tokens`, `num_scheduled_tokens`, `block_table`, `ref_count`, and `Sequence.status` moving in lockstep across prefill, decode, preemption, and postprocess.
- Safe modification: Add unit tests for allocation, prefix cache reuse, chunked prefill, preemption, EOS, and max-token termination before changing scheduling logic.
- Test coverage: No tests detected for scheduler, block manager, or sequence state transitions.

**Global attention context:**
- Files: `nanovllm/utils/context.py`, `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/embed_head.py`
- Why fragile: Attention and LM head depend on module-global `_CONTEXT`, making nested calls, exceptions, async execution, and missed `reset_context()` dangerous.
- Safe modification: Keep context set/reset in `try/finally` and prefer passing context explicitly through model calls where practical.
- Test coverage: No tests detected for context reset after exceptions or mixed prefill/decode calls.

**Tensor-parallel process lifecycle:**
- Files: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/model_runner.py`
- Why fragile: Child processes are spawned before rank 0 model construction completes; shutdown relies on `atexit`, dynamic RPC, barriers, and `join()` without timeout.
- Safe modification: Add explicit `close()` idempotency, join timeouts, child error propagation, and cleanup in failed initialization paths.
- Test coverage: No tensor-parallel startup/shutdown tests detected.

**CUDA-only execution path:**
- Files: `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/sampler.py`
- Why fragile: Import and construction assume CUDA, NCCL, Triton, FlashAttention, and compatible GPU memory; there is no CPU fallback or capability probe.
- Safe modification: Guard construction with explicit environment checks and produce actionable errors for missing CUDA/NCCL/FlashAttention.
- Test coverage: No environment capability tests detected.

**Weight mapping and tensor parallel sharding:**
- Files: `nanovllm/utils/loader.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`, `nanovllm/models/qwen3.py`
- Why fragile: String replacement maps HF names to packed modules; sharding assumes divisibility and exact parameter names.
- Safe modification: Add architecture-specific load tests with tiny safetensors fixtures and explicit missing/unexpected key reporting.
- Test coverage: No loader or sharding tests detected.

## Scaling Limits

**Single-host tensor parallelism only:**
- Current capacity: `Config.tensor_parallel_size` accepts 1–8.
- Limit: Process group rendezvous is hard-coded to local TCP and CUDA device index equals rank.
- Files: `nanovllm/config.py`, `nanovllm/engine/model_runner.py`
- Scaling path: Parameterize distributed init method, rank/device mapping, and multi-node environment variables.

**KV cache capacity is fixed at engine startup:**
- Current capacity: `num_kvcache_blocks` is computed once from available GPU memory and `gpu_memory_utilization`.
- Limit: Memory fragmentation, concurrent workloads, or graph capture changes after startup are not handled dynamically.
- Files: `nanovllm/engine/model_runner.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`
- Scaling path: Track allocation failures with metrics, expose capacity in public API, and support configurable reserve/eviction policies.

**Prompt/output length enforcement is scheduler-local and incomplete:**
- Current capacity: `Config.max_model_len` is applied to HF config and CUDA graph block-table width.
- Limit: Requests are not rejected when prompt length or prompt+completion exceeds `max_model_len`.
- Files: `nanovllm/config.py`, `nanovllm/engine/llm_engine.py`, `nanovllm/engine/model_runner.py`, `nanovllm/engine/scheduler.py`
- Scaling path: Validate every request against max model length before enqueue and return structured errors.

**Shared-memory RPC payload size is fixed:**
- Current capacity: 1 MiB serialized command buffer.
- Limit: Large batches or long sequences can exceed the buffer because full `Sequence` state is pickled for worker ranks.
- Files: `nanovllm/engine/model_runner.py`, `nanovllm/engine/sequence.py`
- Scaling path: Send compact tensor metadata and shared buffers rather than pickled Python sequence objects.

## Dependencies at Risk

**`flash-attn`:**
- Risk: CUDA/PyTorch/platform-sensitive binary dependency with no version upper bound.
- Impact: Installation or import failures break `nanovllm/layers/attention.py` and prevent inference.
- Migration plan: Pin compatible versions per PyTorch/CUDA matrix, make FlashAttention availability checks explicit, and document supported platforms in `pyproject.toml` and `README.md`.

**`torch`, `triton`, and `transformers`:**
- Risk: Lower-bounded dependencies can resolve to incompatible major/minor combinations; `hf_config.dtype` and Qwen3 config assumptions are version-sensitive.
- Impact: Model construction in `nanovllm/engine/model_runner.py` and `nanovllm/models/qwen3.py` can fail after resolver upgrades.
- Migration plan: Add tested upper bounds or a lockfile for development, plus CI against the supported version matrix.

**Direct imports relying on transitive packages:**
- Risk: `numpy`, `safetensors`, and `tqdm` are imported directly but not declared directly in `pyproject.toml`.
- Impact: Packaging metadata under-declares runtime requirements for `nanovllm/engine/block_manager.py`, `nanovllm/utils/loader.py`, and `nanovllm/engine/llm_engine.py`.
- Migration plan: Add direct dependencies for every direct import used at runtime.

**`torch.compile` decorators:**
- Risk: Compile behavior varies across PyTorch versions, device capability, and dynamic shapes.
- Impact: Failures affect `nanovllm/layers/activation.py`, `nanovllm/layers/layernorm.py`, `nanovllm/layers/rotary_embedding.py`, and `nanovllm/layers/sampler.py`.
- Migration plan: Gate compile with config, provide eager fallback, and include compile-disabled tests.

## Missing Critical Features

**Structured error reporting and lifecycle API:**
- Problem: Public usage has `generate` and implicit `atexit` cleanup but no explicit, idempotent `close()`/context-manager contract.
- Files: `nanovllm/llm.py`, `nanovllm/engine/llm_engine.py`, `example.py`
- Blocks: Reliable embedding in services, notebooks, repeated tests, and applications that need deterministic GPU/process cleanup.

**Sampling controls beyond temperature:**
- Problem: No top-p, top-k, min-p, repetition penalty, stop strings, stop token IDs, seed control, or greedy path.
- Files: `nanovllm/sampling_params.py`, `nanovllm/layers/sampler.py`, `nanovllm/engine/scheduler.py`
- Blocks: vLLM-like compatibility and many application-level generation policies.

**Streaming, cancellation, and per-request status:**
- Problem: `generate` batches all requests and returns only after every sequence finishes.
- Files: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/sequence.py`
- Blocks: Interactive serving, request cancellation, timeouts, and partial output delivery.

**Model architecture registry:**
- Problem: Engine construction imports `Qwen3ForCausalLM` directly.
- Files: `nanovllm/engine/model_runner.py`, `nanovllm/models/qwen3.py`
- Blocks: Loading non-Qwen3 architectures without editing core engine code.

**Observability and diagnostics:**
- Problem: No structured logging, counters, capacity metrics, or scheduler/model-runner debug hooks.
- Files: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/model_runner.py`
- Blocks: Production debugging of throughput drops, OOM risk, prefix cache hit rates, preemption rates, and worker failures.

**Reproducible development environment:**
- Problem: No lockfile, no dev extras, no test command, and no CI configuration.
- Files: `pyproject.toml`, `README.md`
- Blocks: Reproducing benchmark results and validating dependency upgrades consistently.

## Test Coverage Gaps

**Scheduler and block manager behavior:**
- What's not tested: Allocation/deallocation, prefix cache reuse, chunked prefill, preemption, EOS handling, and max-token termination.
- Files: `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`, `nanovllm/engine/sequence.py`
- Risk: Token loss, duplicate KV blocks, leaked blocks, starvation, or non-terminating requests can ship unnoticed.
- Priority: High

**Public API validation:**
- What's not tested: Empty prompts, sampling parameter length mismatch, invalid `max_tokens`, invalid model path, excessive prompt length, and output schema.
- Files: `nanovllm/engine/llm_engine.py`, `nanovllm/sampling_params.py`, `nanovllm/config.py`
- Risk: User-facing crashes and silent request drops remain undetected.
- Priority: High

**Model runner lifecycle and distributed RPC:**
- What's not tested: Tensor-parallel startup, fixed port/shared-memory collision, payload sizing, worker exceptions, shutdown idempotency, and `atexit` cleanup.
- Files: `nanovllm/engine/model_runner.py`, `nanovllm/engine/llm_engine.py`
- Risk: Hung processes, leaked shared memory, and unrecoverable NCCL failures.
- Priority: High

**Attention/KV cache integration:**
- What's not tested: `slot_mapping`, `block_tables`, prefix-cache prefill, decode attention, and context reset across failures.
- Files: `nanovllm/layers/attention.py`, `nanovllm/utils/context.py`, `nanovllm/engine/model_runner.py`
- Risk: Incorrect logits, cache corruption, or CUDA graph replay with stale context.
- Priority: High

**Weight loading and tensor parallel sharding:**
- What's not tested: Packed QKV/gate-up mappings, row/column parallel sharding, tied embeddings, missing keys, unexpected keys, and shape mismatches.
- Files: `nanovllm/utils/loader.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`, `nanovllm/models/qwen3.py`
- Risk: Incorrect model weights or hard-to-debug load-time failures.
- Priority: High

**Dependency/platform compatibility:**
- What's not tested: CUDA/NCCL/Triton/FlashAttention availability, `torch.compile` fallback, Python 3.10–3.12 compatibility, and supported PyTorch/Transformers combinations.
- Files: `pyproject.toml`, `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/sampler.py`
- Risk: Installation succeeds but runtime fails on unsupported platform combinations.
- Priority: Medium

---

*Concerns audit: 2026-05-11*
