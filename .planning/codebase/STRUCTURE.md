# Codebase Structure

**Analysis Date:** 2026-05-11

## Directory Layout

```text
nano-vllm/
├── nanovllm/                 # Installable Python package and inference implementation
│   ├── __init__.py           # Public package exports
│   ├── llm.py                # Thin public LLM class
│   ├── config.py             # Runtime configuration dataclass
│   ├── sampling_params.py    # Per-request sampling dataclass
│   ├── engine/               # Engine orchestration, scheduling, sequence state, KV-cache management, model runner
│   │   ├── llm_engine.py     # User-facing generation loop and tensor-parallel process startup
│   │   ├── model_runner.py   # CUDA/distributed model execution backend
│   │   ├── scheduler.py      # Prefill/decode scheduler and completion postprocessing
│   │   ├── block_manager.py  # KV-cache block allocator and prefix-cache metadata
│   │   └── sequence.py       # Request state model
│   ├── models/               # Supported model architecture definitions
│   │   └── qwen3.py          # Qwen3 transformer implementation and packed-weight mapping
│   ├── layers/               # Reusable PyTorch/Triton/FlashAttention layer primitives
│   │   ├── activation.py
│   │   ├── attention.py
│   │   ├── embed_head.py
│   │   ├── layernorm.py
│   │   ├── linear.py
│   │   ├── rotary_embedding.py
│   │   └── sampler.py
│   └── utils/                # Runtime context and weight-loading helpers
│       ├── context.py
│       └── loader.py
├── example.py                # Example inference script
├── bench.py                  # Benchmark script
├── pyproject.toml            # Build metadata and dependencies
├── README.md                 # Project overview, quick start, benchmark notes
├── LICENSE                   # MIT license
├── assets/                   # README assets
│   └── logo.png
├── .claude/skills/           # Project-local GitNexus skill guidance
├── .gitnexus/                # Generated GitNexus index data
├── .omx/                     # OMX runtime state/logs
└── .planning/codebase/       # Generated codebase map documents
```

## Directory Purposes

**`nanovllm/`:**
- Purpose: Contains the installable package and all runtime inference code.
- Contains: Public API files `nanovllm/__init__.py`, `nanovllm/llm.py`, configuration in `nanovllm/config.py`, request options in `nanovllm/sampling_params.py`, and implementation subpackages.
- Key files: `nanovllm/__init__.py`, `nanovllm/llm.py`, `nanovllm/config.py`, `nanovllm/sampling_params.py`.

**`nanovllm/engine/`:**
- Purpose: Houses the engine layer that turns user prompts into scheduled GPU inference work.
- Contains: Engine facade, scheduler, sequence state, cache-block management, and CUDA/distributed model runner.
- Key files: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/model_runner.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`, `nanovllm/engine/sequence.py`.

**`nanovllm/models/`:**
- Purpose: Contains supported transformer architecture implementations.
- Contains: Model classes and Hugging Face weight-name packing maps.
- Key files: `nanovllm/models/qwen3.py`.

**`nanovllm/layers/`:**
- Purpose: Contains reusable neural-network layers and GPU kernels shared by model definitions.
- Contains: Tensor-parallel linear layers, embeddings/LM head, attention, RoPE, RMSNorm, activation, and sampling.
- Key files: `nanovllm/layers/linear.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/embed_head.py`, `nanovllm/layers/layernorm.py`, `nanovllm/layers/rotary_embedding.py`, `nanovllm/layers/activation.py`, `nanovllm/layers/sampler.py`.

**`nanovllm/utils/`:**
- Purpose: Contains support utilities that are shared across engine and layer code.
- Contains: Process-local forward-pass context and safetensors weight loader.
- Key files: `nanovllm/utils/context.py`, `nanovllm/utils/loader.py`.

**Repository root scripts:**
- Purpose: Provide direct usage and benchmark entry points.
- Contains: `example.py` for text generation and `bench.py` for throughput measurement.
- Key files: `example.py`, `bench.py`.

**Repository root metadata:**
- Purpose: Define installation, dependency, licensing, and project documentation metadata.
- Contains: `pyproject.toml`, `README.md`, `LICENSE`, `.gitignore`.
- Key files: `pyproject.toml`, `README.md`, `LICENSE`, `.gitignore`.

**`assets/`:**
- Purpose: Stores static assets referenced by project documentation.
- Contains: Project logo.
- Key files: `assets/logo.png`.

**`.claude/skills/`:**
- Purpose: Stores project-local GitNexus workflow guidance for code exploration, impact analysis, debugging, refactoring, and indexing.
- Contains: Skill markdown under `.claude/skills/gitnexus/`.
- Key files: `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md`, `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md`, `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md`, `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md`, `.claude/skills/gitnexus/gitnexus-cli/SKILL.md`, `.claude/skills/gitnexus/gitnexus-guide/SKILL.md`.

**`.planning/codebase/`:**
- Purpose: Stores generated GSD codebase mapping documents.
- Contains: Architecture and structure documents in this mapping pass.
- Key files: `.planning/codebase/ARCHITECTURE.md`, `.planning/codebase/STRUCTURE.md`.

## Key File Locations

**Entry Points:**
- `nanovllm/__init__.py`: Public imports for `LLM` and `SamplingParams`.
- `nanovllm/llm.py`: Public `LLM` class that inherits from `LLMEngine`.
- `example.py`: End-user script for prompt formatting, generation, and printing completions.
- `bench.py`: Benchmark script for randomized prompt-token workloads and throughput reporting.
- `nanovllm/engine/model_runner.py`: Tensor-parallel child-process target through `Process(target=ModelRunner, ...)`.

**Configuration:**
- `pyproject.toml`: Build backend, package metadata, Python version range, runtime dependencies, and package discovery.
- `nanovllm/config.py`: Runtime engine configuration and Hugging Face config loading.
- `nanovllm/sampling_params.py`: Per-request sampling configuration.
- `.gitignore`: Ignore rules for Python caches, build outputs, virtual environments, `.env`, `.pypirc`, and editor/tool caches.

**Core Logic:**
- `nanovllm/engine/llm_engine.py`: Top-level generation lifecycle, request admission, progress display, and output decoding.
- `nanovllm/engine/scheduler.py`: Prefill/decode scheduling, preemption, and postprocess completion logic.
- `nanovllm/engine/block_manager.py`: KV-cache block allocation, deallocation, prefix-cache hashing, and reference counting.
- `nanovllm/engine/sequence.py`: Mutable request state and token/block-table accessors.
- `nanovllm/engine/model_runner.py`: CUDA setup, distributed setup, model loading, KV-cache allocation, tensor preparation, sampling, shared-memory RPC, and CUDA graph capture.
- `nanovllm/models/qwen3.py`: Qwen3 transformer model, decoder layer, attention/MLP composition, and packed-module weight mapping.
- `nanovllm/layers/attention.py`: Triton KV-cache write kernel plus FlashAttention prefill/decode calls.
- `nanovllm/layers/linear.py`: Tensor-parallel linear abstractions and weight sharding.
- `nanovllm/layers/embed_head.py`: Tensor-parallel embedding and LM-head behavior.
- `nanovllm/utils/context.py`: Forward-pass context shared by runner and layers.
- `nanovllm/utils/loader.py`: Safetensors model loading and parameter-specific loader dispatch.

**Testing:**
- Not detected: no `tests/`, `test_*.py`, `*_test.py`, or test configuration files are present in the repository root or `nanovllm/`.
- Runtime examples: `example.py` and `bench.py` act as executable smoke/benchmark scripts rather than automated tests.

**Generated/Runtime State:**
- `.gitnexus/`: Generated GitNexus graph/index data.
- `.omx/`: OMX runtime state and logs.
- `.planning/codebase/`: Generated architecture/structure mapping documents.

## Naming Conventions

**Files:**
- Use snake_case Python module names for engine files: `nanovllm/engine/llm_engine.py`, `nanovllm/engine/model_runner.py`, `nanovllm/engine/block_manager.py`.
- Use snake_case Python module names for layer files: `nanovllm/layers/rotary_embedding.py`, `nanovllm/layers/embed_head.py`.
- Use lowercase architecture identifiers for model files: `nanovllm/models/qwen3.py`.
- Use root-level script names for executable examples: `example.py`, `bench.py`.

**Directories:**
- Use package subdirectories by responsibility: `nanovllm/engine/`, `nanovllm/models/`, `nanovllm/layers/`, `nanovllm/utils/`.
- Keep public package root files in `nanovllm/` and implementation-heavy code in subpackages such as `nanovllm/engine/` and `nanovllm/layers/`.
- Keep generated planning artifacts under `.planning/codebase/`.

**Classes and Types:**
- Use PascalCase for classes: `LLMEngine`, `ModelRunner`, `Scheduler`, `BlockManager`, `Sequence`, `Config`, `SamplingParams`, `Qwen3ForCausalLM`, `RMSNorm`, `Sampler`.
- Use dataclasses for compact configuration/state carriers: `Config` in `nanovllm/config.py`, `SamplingParams` in `nanovllm/sampling_params.py`, `Context` in `nanovllm/utils/context.py`.
- Use enum classes for finite state: `SequenceStatus` in `nanovllm/engine/sequence.py`.

**Functions and Methods:**
- Use snake_case for functions and methods: `add_request`, `prepare_prefill`, `prepare_decode`, `allocate_kv_cache`, `capture_cudagraph`, `load_model`, `get_context`.
- Use leading underscores for private helper methods: `_allocate_block` and `_deallocate_block` in `nanovllm/engine/block_manager.py`.
- Use concise kernel/helper names near their domain: `store_kvcache_kernel` and `store_kvcache` in `nanovllm/layers/attention.py`.

**Module Constants and Shared State:**
- Use uppercase for module-global process-local context storage: `_CONTEXT` in `nanovllm/utils/context.py`.
- Use class attributes for shared defaults and counters: `Sequence.block_size` and `Sequence.counter` in `nanovllm/engine/sequence.py`.
- Use lower_snake_case class attributes for mapping tables: `packed_modules_mapping` in `nanovllm/models/qwen3.py`.

## Where to Add New Code

**New Feature:**
- Primary code: place engine behavior in `nanovllm/engine/` when it affects request scheduling, cache management, model execution, or generation orchestration.
- Tests: Not detected; introduce a `tests/` directory only when adding an automated test convention for the repository.
- Public API updates: expose user-facing classes/functions through `nanovllm/__init__.py` and keep compatibility wrappers thin in `nanovllm/llm.py`.

**New Configuration Option:**
- Primary code: add engine-wide runtime options to `Config` in `nanovllm/config.py`.
- Integration point: rely on `LLMEngine.__init__` in `nanovllm/engine/llm_engine.py` to forward keyword arguments matching `Config` dataclass fields.

**New Sampling Option:**
- Primary code: add request-level options to `SamplingParams` in `nanovllm/sampling_params.py`.
- Engine state: copy new scalar request state into `Sequence` in `nanovllm/engine/sequence.py`.
- GPU sampling logic: update `Sampler` in `nanovllm/layers/sampler.py` and `ModelRunner.prepare_sample` in `nanovllm/engine/model_runner.py`.

**New Scheduler or KV-Cache Behavior:**
- Primary code: update scheduling policy in `nanovllm/engine/scheduler.py`.
- Cache metadata: update block allocation, prefix hashing, or ref-counting in `nanovllm/engine/block_manager.py`.
- Sequence fields: add request/cache state to `nanovllm/engine/sequence.py`.

**New Model Architecture:**
- Implementation: add a new model module under `nanovllm/models/`, following the composition pattern in `nanovllm/models/qwen3.py`.
- Runner integration: update model construction/import selection in `nanovllm/engine/model_runner.py`, which currently instantiates `Qwen3ForCausalLM`.
- Weight loading: add architecture-specific packed-weight mapping on the model class and parameter `weight_loader` hooks in layer modules as needed.

**New Component/Module:**
- Implementation: place reusable neural-network primitives in `nanovllm/layers/`.
- Engine-facing implementation: place runtime orchestration components in `nanovllm/engine/`.
- Model-specific implementation: place transformer/model composition in `nanovllm/models/`.

**Utilities:**
- Shared helpers: place context, file loading, or generic runtime utilities in `nanovllm/utils/`.
- Keep layer-specific helpers next to their layer file, such as `store_kvcache` in `nanovllm/layers/attention.py`.

**New Script or Manual Workflow:**
- Root-level examples: add small executable scripts beside `example.py` and `bench.py` when they demonstrate package usage.
- Documentation assets: place static documentation assets in `assets/`.
- Project documentation: update `README.md` for user-facing instructions and keep generated codebase maps under `.planning/codebase/`.

## Special Directories

**`.planning/codebase/`:**
- Purpose: Generated codebase analysis documents used by GSD planning/execution workflows.
- Generated: Yes.
- Committed: No tracked files detected under `.planning/` at analysis time.

**`.gitnexus/`:**
- Purpose: Generated GitNexus code-intelligence index for repository graph queries.
- Generated: Yes.
- Committed: No tracked files detected under `.gitnexus/` at analysis time.

**`.omx/`:**
- Purpose: OMX runtime state and logs for orchestration workflows.
- Generated: Yes.
- Committed: No tracked files detected under `.omx/` at analysis time.

**`.claude/skills/`:**
- Purpose: Project-local skill guidance, currently GitNexus workflow docs.
- Generated: Yes.
- Committed: No tracked files detected under `.claude/` at analysis time.

**`assets/`:**
- Purpose: Static documentation assets.
- Generated: No.
- Committed: Yes; `assets/logo.png` is tracked.

**`nanovllm/`:**
- Purpose: Installable package source.
- Generated: No.
- Committed: Yes; package files under `nanovllm/` are tracked.

---

*Structure analysis: 2026-05-11*
