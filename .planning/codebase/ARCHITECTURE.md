# Architecture

**Analysis Date:** 2026-05-11

## Pattern Overview

**Overall:** Layered offline LLM inference library with a thin public API, request scheduler, GPU model runner, Qwen3 model implementation, and reusable PyTorch/Triton layer primitives.

**Key Characteristics:**
- Public API is intentionally minimal: `nanovllm/__init__.py` exports `LLM` and `SamplingParams`, while `nanovllm/llm.py` keeps `LLM` as a thin subclass of `nanovllm.engine.llm_engine.LLMEngine`.
- Runtime orchestration is centralized in `nanovllm/engine/llm_engine.py`, which owns configuration creation, tokenizer loading, tensor-parallel worker startup, request admission, generation progress, and shutdown.
- Scheduling and KV-cache ownership are separate from model execution: `nanovllm/engine/scheduler.py` chooses prefill/decode batches, and `nanovllm/engine/block_manager.py` manages cache blocks and prefix-cache reuse.
- GPU execution is isolated behind `nanovllm/engine/model_runner.py`, which initializes distributed state, loads `nanovllm/models/qwen3.py`, allocates KV cache, captures decode CUDA graphs, and executes `run`.
- Model architecture is separated from runtime policy: `nanovllm/models/qwen3.py` composes Qwen3 modules from primitives in `nanovllm/layers/`.
- Layer primitives in `nanovllm/layers/` encapsulate tensor parallel sharding, FlashAttention invocation, Triton KV-cache writes, RoPE, RMSNorm, activation, embeddings, and sampling.
- Per-forward-pass attention metadata is passed through the lightweight process-local context in `nanovllm/utils/context.py`.

## Layers

**Public API Layer:**
- Purpose: Provide the user-facing import surface and vLLM-like construction/generation API.
- Location: `nanovllm/__init__.py`, `nanovllm/llm.py`, `nanovllm/sampling_params.py`
- Contains: Public exports, `LLM`, and `SamplingParams`.
- Depends on: `nanovllm.engine.llm_engine.LLMEngine` and Python dataclasses.
- Used by: `example.py`, `bench.py`, and package consumers importing `from nanovllm import LLM, SamplingParams`.

**Configuration Layer:**
- Purpose: Normalize runtime options and load Hugging Face model configuration.
- Location: `nanovllm/config.py`
- Contains: `Config` dataclass with model path, batch limits, sequence limits, tensor-parallel size, eager/CUDA-graph flag, KV-cache block settings, and Hugging Face config.
- Depends on: `transformers.AutoConfig` and filesystem validation via `os.path.isdir`.
- Used by: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, and `nanovllm/engine/model_runner.py`.

**Engine Orchestration Layer:**
- Purpose: Convert API calls into scheduled inference work and decoded outputs.
- Location: `nanovllm/engine/llm_engine.py`
- Contains: `LLMEngine.__init__`, `add_request`, `step`, `is_finished`, `generate`, and `exit`.
- Depends on: `Config`, `SamplingParams`, `Sequence`, `Scheduler`, `ModelRunner`, `AutoTokenizer`, `torch.multiprocessing`, `tqdm`, and `atexit`.
- Used by: `nanovllm/llm.py`, `example.py`, and `bench.py`.

**Request State Layer:**
- Purpose: Represent each prompt/completion request and its mutable generation/cache state.
- Location: `nanovllm/engine/sequence.py`
- Contains: `SequenceStatus`, `Sequence`, token lists, block tables, prompt/completion views, scheduled-token counters, and multiprocessing pickle state.
- Depends on: `nanovllm.sampling_params.SamplingParams`.
- Used by: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`, and `nanovllm/engine/model_runner.py`.

**Scheduling and Cache-Block Layer:**
- Purpose: Decide which sequences run next, manage waiting/running queues, preempt decode work when cache space is low, and map sequences to KV-cache blocks.
- Location: `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`
- Contains: `Scheduler`, `BlockManager`, `Block`, waiting/running deques, prefix-cache hash table, free/used block sets, allocation/deallocation logic, and postprocess logic.
- Depends on: `Config`, `Sequence`, `SequenceStatus`, `xxhash`, and `numpy`.
- Used by: `nanovllm/engine/llm_engine.py` for request scheduling and completion updates.

**Model Execution Layer:**
- Purpose: Own CUDA device setup, distributed tensor-parallel communication, model loading, KV-cache tensor allocation, prefill/decode tensor preparation, sampling, and CUDA graph replay.
- Location: `nanovllm/engine/model_runner.py`
- Contains: `ModelRunner`, shared-memory RPC methods, `warmup_model`, `allocate_kv_cache`, `prepare_prefill`, `prepare_decode`, `prepare_sample`, `run_model`, `run`, and `capture_cudagraph`.
- Depends on: `torch`, `torch.distributed`, multiprocessing `Event`, `SharedMemory`, `Qwen3ForCausalLM`, `Sampler`, context utilities, and `load_model`.
- Used by: `nanovllm/engine/llm_engine.py` as the synchronous rank-0 runner and as the target for tensor-parallel child processes.

**Model Definition Layer:**
- Purpose: Define the supported transformer architecture and packed-weight mapping.
- Location: `nanovllm/models/qwen3.py`
- Contains: `Qwen3Attention`, `Qwen3MLP`, `Qwen3DecoderLayer`, `Qwen3Model`, `Qwen3ForCausalLM`, and `packed_modules_mapping`.
- Depends on: `torch`, `torch.distributed`, `transformers.Qwen3Config`, and primitives from `nanovllm/layers/`.
- Used by: `nanovllm/engine/model_runner.py`.

**Layer Primitive and Kernel Layer:**
- Purpose: Implement reusable neural-network building blocks and GPU kernels.
- Location: `nanovllm/layers/activation.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/embed_head.py`, `nanovllm/layers/layernorm.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/rotary_embedding.py`, `nanovllm/layers/sampler.py`
- Contains: `SiluAndMul`, `Attention`, `store_kvcache_kernel`, `VocabParallelEmbedding`, `ParallelLMHead`, `RMSNorm`, tensor-parallel linear classes, `RotaryEmbedding`, `get_rope`, and `Sampler`.
- Depends on: `torch`, `torch.nn`, `torch.distributed`, `triton`, `flash_attn`, and `nanovllm.utils.context`.
- Used by: `nanovllm/models/qwen3.py` and `nanovllm/engine/model_runner.py`.

**Utility Layer:**
- Purpose: Provide cross-cutting runtime helpers for context propagation and weight loading.
- Location: `nanovllm/utils/context.py`, `nanovllm/utils/loader.py`
- Contains: `Context`, `get_context`, `set_context`, `reset_context`, `default_weight_loader`, and `load_model`.
- Depends on: `torch`, `safetensors.safe_open`, and filesystem globbing.
- Used by: `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/embed_head.py`, and `nanovllm/models/qwen3.py`.

**Script Layer:**
- Purpose: Provide executable examples for local inference and benchmark runs.
- Location: `example.py`, `bench.py`
- Contains: `main()` functions that instantiate `LLM`, build `SamplingParams`, and call `generate`.
- Depends on: `nanovllm`, `transformers.AutoTokenizer`, standard-library timing/random utilities, and a local Hugging Face model directory.
- Used by: Developers running the project directly.

## Data Flow

**User-facing generation flow:**

1. Caller imports `LLM` and `SamplingParams` from `nanovllm/__init__.py` and instantiates `LLM` through `nanovllm/llm.py`.
2. `LLMEngine.__init__` in `nanovllm/engine/llm_engine.py` builds `Config` from `nanovllm/config.py`, sets `Sequence.block_size` in `nanovllm/engine/sequence.py`, starts tensor-parallel `ModelRunner` processes from `nanovllm/engine/model_runner.py`, creates rank-0 `ModelRunner`, loads `AutoTokenizer`, creates `Scheduler`, and registers `exit`.
3. `LLMEngine.generate` in `nanovllm/engine/llm_engine.py` normalizes `SamplingParams` from `nanovllm/sampling_params.py`, calls `add_request`, then loops over `step` until `is_finished`.
4. `LLMEngine.add_request` in `nanovllm/engine/llm_engine.py` tokenizes string prompts with the tokenizer, creates `Sequence` objects from `nanovllm/engine/sequence.py`, and submits them to `Scheduler.add` in `nanovllm/engine/scheduler.py`.
5. `Scheduler.schedule` in `nanovllm/engine/scheduler.py` returns scheduled sequences plus an `is_prefill` flag; `LLMEngine.step` sends that batch to `ModelRunner.call("run", seqs, is_prefill)` in `nanovllm/engine/model_runner.py`.
6. `ModelRunner.run` in `nanovllm/engine/model_runner.py` prepares input tensors and context, executes `Qwen3ForCausalLM` from `nanovllm/models/qwen3.py`, samples next tokens with `Sampler` from `nanovllm/layers/sampler.py`, resets `Context`, and returns token IDs on rank 0.
7. `Scheduler.postprocess` in `nanovllm/engine/scheduler.py` updates cache hashes through `BlockManager.hash_blocks`, appends generated tokens to `Sequence`, detects EOS/max-token completion, deallocates cache blocks, and removes finished sequences.
8. `LLMEngine.generate` in `nanovllm/engine/llm_engine.py` collects finished `completion_token_ids`, decodes them with the tokenizer, and returns dictionaries containing `text` and `token_ids`.

**Prefill and prefix-cache flow:**

1. Waiting sequences enter `Scheduler.schedule` in `nanovllm/engine/scheduler.py`.
2. `BlockManager.can_allocate` in `nanovllm/engine/block_manager.py` hashes full prompt blocks with `compute_hash`, checks `hash_to_block_id`, and reports reusable cached blocks.
3. `BlockManager.allocate` in `nanovllm/engine/block_manager.py` fills `Sequence.block_table` with reused and newly allocated block IDs.
4. `ModelRunner.prepare_prefill` in `nanovllm/engine/model_runner.py` builds `input_ids`, `positions`, cumulative sequence lengths, slot mapping, and optional block tables for prefix-cache attention.
5. `set_context` in `nanovllm/utils/context.py` stores prefill metadata for `Attention.forward` in `nanovllm/layers/attention.py`.
6. `Attention.forward` in `nanovllm/layers/attention.py` writes keys/values into the allocated KV cache with `store_kvcache` and runs `flash_attn_varlen_func`; when `Context.block_tables` is present, FlashAttention reads cached prefix blocks.

**Decode flow:**

1. Running sequences enter the decode branch of `Scheduler.schedule` in `nanovllm/engine/scheduler.py`.
2. `BlockManager.can_append` and `BlockManager.may_append` in `nanovllm/engine/block_manager.py` ensure each sequence has room for the next token, while `Scheduler.preempt` moves cache-starved sequences back to waiting.
3. `ModelRunner.prepare_decode` in `nanovllm/engine/model_runner.py` builds one-token `input_ids`, positions, slot mapping, context lengths, and block tables, then calls `set_context` in `nanovllm/utils/context.py`.
4. `ModelRunner.run_model` in `nanovllm/engine/model_runner.py` uses normal eager execution for prefill/eager/large decode batches and uses captured CUDA graphs for decode batches when `Config.enforce_eager` from `nanovllm/config.py` is false and batch size is within graph limits.
5. `Attention.forward` in `nanovllm/layers/attention.py` runs `flash_attn_with_kvcache` against the allocated KV-cache tensors.

**Tensor-parallel control flow:**

1. `LLMEngine.__init__` in `nanovllm/engine/llm_engine.py` starts one child process per nonzero rank using `mp.get_context("spawn")` and `Process(target=ModelRunner, args=(config, i, event))`.
2. Every `ModelRunner` in `nanovllm/engine/model_runner.py` calls `dist.init_process_group("nccl", "tcp://localhost:2333", world_size=..., rank=...)` and selects its CUDA device by rank.
3. Rank 0 creates `SharedMemory(name="nanovllm")`; nonzero ranks attach to it and enter `ModelRunner.loop` in `nanovllm/engine/model_runner.py`.
4. Rank 0 `ModelRunner.call` in `nanovllm/engine/model_runner.py` serializes `[method_name, *args]` into shared memory through `write_shm`, signals child events, and executes the local rank-0 method.
5. Nonzero ranks call `read_shm`, dispatch the named method with `call`, and exit the loop when the method name is `exit`.

**Weight-loading flow:**

1. `ModelRunner.__init__` in `nanovllm/engine/model_runner.py` creates `Qwen3ForCausalLM` from `nanovllm/models/qwen3.py`.
2. `load_model` in `nanovllm/utils/loader.py` scans `*.safetensors` files under `Config.model` from `nanovllm/config.py`.
3. `Qwen3ForCausalLM.packed_modules_mapping` in `nanovllm/models/qwen3.py` maps Hugging Face Q/K/V and gate/up weights into packed local module names.
4. Layer-specific `weight_loader` methods in `nanovllm/layers/linear.py` and `nanovllm/layers/embed_head.py` shard or copy tensors according to the tensor-parallel rank.

**State Management:**
- Request lifecycle state is stored on `Sequence` instances in `nanovllm/engine/sequence.py`.
- Scheduling queues are stored in `Scheduler.waiting` and `Scheduler.running` in `nanovllm/engine/scheduler.py`.
- KV-cache block metadata is stored in `BlockManager.blocks`, `hash_to_block_id`, `free_block_ids`, and `used_block_ids` in `nanovllm/engine/block_manager.py`.
- CUDA KV-cache tensor storage is allocated in `ModelRunner.allocate_kv_cache` and attached to attention modules in `nanovllm/engine/model_runner.py`.
- Forward-pass metadata is held in process-local `_CONTEXT` in `nanovllm/utils/context.py`; call `reset_context` after each model run.

## Key Abstractions

**`LLM` / `LLMEngine`:**
- Purpose: Public inference engine and orchestration facade.
- Examples: `nanovllm/llm.py`, `nanovllm/engine/llm_engine.py`
- Pattern: Keep `LLM` thin; place runtime orchestration methods on `LLMEngine`.

**`Config`:**
- Purpose: Centralize model path, capacity limits, tensor-parallel settings, eager/CUDA-graph mode, and Hugging Face config.
- Examples: `nanovllm/config.py`
- Pattern: Add engine-wide runtime options as dataclass fields in `Config`; `LLMEngine.__init__` in `nanovllm/engine/llm_engine.py` automatically forwards matching keyword arguments.

**`SamplingParams`:**
- Purpose: Carry per-request sampling behavior into `Sequence` and `Sampler`.
- Examples: `nanovllm/sampling_params.py`, `nanovllm/engine/sequence.py`, `nanovllm/layers/sampler.py`
- Pattern: Keep request-level generation options compact and copy scalar values onto each `Sequence`.

**`Sequence`:**
- Purpose: Represent one request, including prompt tokens, generated tokens, cache table, scheduled-token counters, and pickle-friendly state.
- Examples: `nanovllm/engine/sequence.py`
- Pattern: Use `Sequence` as mutable engine state; scheduler, block manager, and runner all read/write it.

**`Scheduler`:**
- Purpose: Batch prefill/decode work, enforce max sequence/token limits, apply preemption, and finish sequences.
- Examples: `nanovllm/engine/scheduler.py`
- Pattern: Keep policy decisions in `Scheduler.schedule` and completion side effects in `Scheduler.postprocess`.

**`BlockManager`:**
- Purpose: Allocate KV-cache block IDs, share prefix-cache blocks, and manage block reference counts.
- Examples: `nanovllm/engine/block_manager.py`
- Pattern: Keep cache metadata on CPU objects and expose only block tables to GPU execution.

**`ModelRunner`:**
- Purpose: Boundary between engine policy and CUDA/distributed model execution.
- Examples: `nanovllm/engine/model_runner.py`
- Pattern: All model execution goes through `ModelRunner.call`; all batch preparation goes through `prepare_prefill` or `prepare_decode`.

**`Context`:**
- Purpose: Provide attention and LM-head layers with per-batch metadata without threading many arguments through every model layer.
- Examples: `nanovllm/utils/context.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/embed_head.py`
- Pattern: Set context immediately before model execution and reset it immediately after `ModelRunner.run`.

**`Qwen3ForCausalLM` / `Qwen3Model`:**
- Purpose: Supported transformer model definition and logits computation.
- Examples: `nanovllm/models/qwen3.py`
- Pattern: Keep model composition in `nanovllm/models/qwen3.py` and keep reusable math/distributed primitives in `nanovllm/layers/`.

**Tensor-parallel layer classes:**
- Purpose: Encapsulate weight sharding and distributed reductions/gathers.
- Examples: `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`
- Pattern: Attach a `weight_loader` callable to each parameter so `load_model` in `nanovllm/utils/loader.py` stays generic.

**Attention kernels:**
- Purpose: Bridge context metadata, KV-cache tensors, Triton cache writes, and FlashAttention calls.
- Examples: `nanovllm/layers/attention.py`
- Pattern: Keep prefill/decode branching inside `Attention.forward` and derive behavior from `Context`.

## Entry Points

**Package import API:**
- Location: `nanovllm/__init__.py`
- Triggers: `from nanovllm import LLM, SamplingParams`
- Responsibilities: Re-export public API classes from `nanovllm/llm.py` and `nanovllm/sampling_params.py`.

**Public engine class:**
- Location: `nanovllm/llm.py`
- Triggers: `LLM(...)` from external callers, `example.py`, and `bench.py`
- Responsibilities: Present the public class name while inheriting implementation from `nanovllm/engine/llm_engine.py`.

**Interactive/example script:**
- Location: `example.py`
- Triggers: `python example.py`
- Responsibilities: Load tokenizer, apply chat templates, run `LLM.generate`, and print completions.

**Benchmark script:**
- Location: `bench.py`
- Triggers: `python bench.py`
- Responsibilities: Generate randomized token-id prompts and per-request sampling parameters, warm up the engine, measure throughput, and print benchmark metrics.

**Tensor-parallel worker entry:**
- Location: `nanovllm/engine/model_runner.py`
- Triggers: `torch.multiprocessing.Process(target=ModelRunner, ...)` from `nanovllm/engine/llm_engine.py`
- Responsibilities: Initialize rank-local CUDA/distributed runtime and serve method calls from shared memory for nonzero ranks.

**Package metadata/build entry:**
- Location: `pyproject.toml`
- Triggers: Python packaging/build tools.
- Responsibilities: Define project metadata, Python version range, dependencies, and package discovery for `nanovllm*`.

## Error Handling

**Strategy:** Assert-driven precondition checks with cleanup registered through `atexit`; no custom exception hierarchy is defined.

**Patterns:**
- Use dataclass `__post_init__` assertions in `nanovllm/config.py` and `nanovllm/sampling_params.py` for model directory, KV-cache block size, tensor-parallel size, and non-greedy sampling constraints.
- Use internal `assert` statements in `nanovllm/engine/block_manager.py`, `nanovllm/engine/scheduler.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`, and `nanovllm/layers/rotary_embedding.py` to enforce invariants.
- Use `LLMEngine.exit` in `nanovllm/engine/llm_engine.py` and `ModelRunner.exit` in `nanovllm/engine/model_runner.py` to clean up model runners, shared memory, CUDA graphs, CUDA synchronization, and distributed process groups.
- File/model-loading errors propagate from `AutoConfig.from_pretrained` in `nanovllm/config.py`, `AutoTokenizer.from_pretrained` in `nanovllm/engine/llm_engine.py`, and `safe_open`/`get_parameter` in `nanovllm/utils/loader.py`.

## Cross-Cutting Concerns

**Logging:** Progress and throughput are surfaced through `tqdm` in `nanovllm/engine/llm_engine.py`; scripts use `print` in `example.py` and `bench.py`; no structured logging layer is present.

**Validation:** Input/config validation uses dataclass `__post_init__` methods in `nanovllm/config.py` and `nanovllm/sampling_params.py`, plus runtime asserts across `nanovllm/engine/` and `nanovllm/layers/`.

**Authentication:** Not applicable; no auth layer, network server, API route, database, or credential-reading code is present in `nanovllm/`, `example.py`, or `bench.py`.

**Distributed Execution:** Tensor-parallel communication uses `torch.distributed` in `nanovllm/engine/model_runner.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`, and `nanovllm/models/qwen3.py`.

**GPU Performance:** CUDA graph capture lives in `nanovllm/engine/model_runner.py`; FlashAttention and Triton integration live in `nanovllm/layers/attention.py`; `torch.compile` is used in `nanovllm/layers/activation.py`, `nanovllm/layers/layernorm.py`, `nanovllm/layers/rotary_embedding.py`, and `nanovllm/layers/sampler.py`.

**Secrets and Environment:** No `.env` or credential files are present in the repository root; `.gitignore` excludes `.env`, virtual environments, `.pypirc`, caches, and build artifacts.

---

*Architecture analysis: 2026-05-11*
