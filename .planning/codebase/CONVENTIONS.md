# Coding Conventions

**Analysis Date:** 2026-05-11

## Naming Patterns

**Files:**
- Use lowercase snake_case for Python modules: `nanovllm/sampling_params.py`, `nanovllm/engine/block_manager.py`, `nanovllm/layers/rotary_embedding.py`.
- Use short top-level script names for manual entry points: `example.py`, `bench.py`.
- Package directories are lowercase functional areas: `nanovllm/engine/`, `nanovllm/layers/`, `nanovllm/models/`, `nanovllm/utils/`.

**Functions:**
- Use `snake_case` for functions and methods: `nanovllm/engine/model_runner.py` defines `allocate_kv_cache`, `prepare_block_tables`, `prepare_prefill`, `prepare_decode`, and `capture_cudagraph`.
- Use framework-standard method names for PyTorch modules: `forward` appears throughout `nanovllm/layers/linear.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/layernorm.py`, and `nanovllm/models/qwen3.py`.
- Use leading underscores for private helpers: `nanovllm/engine/block_manager.py` defines `_allocate_block` and `_deallocate_block`.
- Use explicit suffixes for specialized kernels and loaders: `store_kvcache_kernel` in `nanovllm/layers/attention.py`, `weight_loader` methods in `nanovllm/layers/linear.py`, and `default_weight_loader` in `nanovllm/utils/loader.py`.

**Variables:**
- Use `snake_case` for regular variables and attributes: `num_cached_tokens`, `num_scheduled_tokens`, `block_table`, `slot_mapping`, and `context_lens` in `nanovllm/engine/sequence.py` and `nanovllm/utils/context.py`.
- Use compact tensor names only in dense math paths: `q`, `k`, `v`, and `o` in `nanovllm/layers/attention.py` and `nanovllm/models/qwen3.py`.
- Use leading-underscore module globals for private process state: `_CONTEXT` in `nanovllm/utils/context.py`.
- Use uppercase enum members for status values: `WAITING`, `RUNNING`, and `FINISHED` in `nanovllm/engine/sequence.py`.

**Types:**
- Use `PascalCase` for classes and dataclasses: `Config` in `nanovllm/config.py`, `SamplingParams` in `nanovllm/sampling_params.py`, `LLMEngine` in `nanovllm/engine/llm_engine.py`, and `Qwen3ForCausalLM` in `nanovllm/models/qwen3.py`.
- Use `PascalCase` plus architecture/provider prefixing for model classes: `Qwen3Attention`, `Qwen3MLP`, `Qwen3DecoderLayer`, and `Qwen3Model` in `nanovllm/models/qwen3.py`.
- Use `dataclass(slots=True)` for lightweight configuration and shared-context records: `Config` in `nanovllm/config.py`, `SamplingParams` in `nanovllm/sampling_params.py`, and `Context` in `nanovllm/utils/context.py`.

## Code Style

**Formatting:**
- Formatter configuration is not detected: no `ruff.toml`, `.ruff.toml`, `.flake8`, `setup.cfg`, `tox.ini`, `pytest.ini`, `.prettierrc`, or formatter sections in `pyproject.toml`.
- Use 4-space indentation, blank lines between top-level definitions, and compact method bodies matching `nanovllm/engine/scheduler.py` and `nanovllm/layers/linear.py`.
- Line length is not mechanically enforced. Existing files include compact long lines in `nanovllm/utils/context.py`, `nanovllm/engine/sequence.py`, `nanovllm/engine/model_runner.py`, and `bench.py`.
- Module docstrings are not used in the package files under `nanovllm/`.

**Linting:**
- Lint configuration is not detected in `pyproject.toml` or standalone lint config files.
- Type annotations are partial and concentrated on public data, tensor boundaries, and return shapes. Examples include `Config` fields in `nanovllm/config.py`, `LLMEngine.generate` in `nanovllm/engine/llm_engine.py`, and tensor methods in `nanovllm/layers/layernorm.py`.
- Modern Python type syntax is used: `AutoConfig | None` in `nanovllm/config.py`, `list[int]` in `nanovllm/engine/sequence.py`, and `tuple[list[Sequence], bool]` in `nanovllm/engine/scheduler.py`.

## Import Organization

**Order:**
1. Standard library imports first: `os`, `dataclasses`, `collections`, `enum`, `itertools`, `time`, `pickle`, and `multiprocessing` in files such as `nanovllm/config.py`, `nanovllm/engine/sequence.py`, and `nanovllm/engine/model_runner.py`.
2. Third-party imports next: `torch`, `triton`, `transformers`, `flash_attn`, `safetensors`, `xxhash`, `numpy`, and `tqdm` in `nanovllm/layers/attention.py`, `nanovllm/models/qwen3.py`, and `nanovllm/utils/loader.py`.
3. Local package imports last with absolute package paths: `from nanovllm.config import Config`, `from nanovllm.engine.sequence import Sequence`, and `from nanovllm.layers.linear import QKVParallelLinear` in `nanovllm/engine/model_runner.py` and `nanovllm/models/qwen3.py`.

**Path Aliases:**
- No path alias system is configured. Use absolute `nanovllm...` imports for package-local references.
- Public API exports are limited to `LLM` and `SamplingParams` in `nanovllm/__init__.py`.

## Error Handling

**Patterns:**
- Use `assert` for internal invariants and shape assumptions: model path and config checks in `nanovllm/config.py`, sampling validation in `nanovllm/sampling_params.py`, tensor stride checks in `nanovllm/layers/attention.py`, and block allocation invariants in `nanovllm/engine/block_manager.py`.
- Use `raise NotImplementedError` for abstract base behavior: `LinearBase.forward` in `nanovllm/layers/linear.py`.
- Custom exception classes are not defined.
- Try/except recovery blocks are not used in package code; lifecycle cleanup is handled through `atexit.register(self.exit)` in `nanovllm/engine/llm_engine.py` and explicit `exit` methods in `nanovllm/engine/model_runner.py`.

## Logging

**Framework:** console/tqdm

**Patterns:**
- No `logging` logger is configured.
- Generation progress is reported with `tqdm` in `nanovllm/engine/llm_engine.py`.
- Manual scripts print user-visible output directly with `print` in `example.py` and `bench.py`.
- Package modules under `nanovllm/` do not print during normal computation.

## Comments

**When to Comment:**
- Comments are sparse and used for phase labels or special execution modes: `# prefill` and `# decode` in `nanovllm/engine/scheduler.py`, `# prefix cache` and `# decode` in `nanovllm/layers/attention.py`, and `# warmup` in `nanovllm/engine/model_runner.py`.
- Comments in `bench.py` document optional vLLM comparison setup.

**JSDoc/TSDoc:**
- Not applicable for this Python codebase.
- Python docstrings are not used in the inspected package modules; behavior is conveyed through small functions, type hints, and file organization.

## Function Design

**Size:** Functions are generally short and direct in layers and utilities. Larger orchestration methods are concentrated in `nanovllm/engine/model_runner.py` for CUDA graph capture, KV cache allocation, prefill/decode preparation, and multiprocessing coordination.

**Parameters:** Prefer explicit scalar/tensor parameters in layer modules, as in `Qwen3Attention.__init__` in `nanovllm/models/qwen3.py` and `store_kvcache` in `nanovllm/layers/attention.py`. Configuration-heavy setup passes `Config` objects into engine components, as in `Scheduler` and `ModelRunner`.

**Return Values:** Use simple return types: tensors from PyTorch modules, token ID lists from `ModelRunner.run`, tuples for scheduler output in `nanovllm/engine/scheduler.py`, and dict-like generation outputs from `LLMEngine.generate` in `nanovllm/engine/llm_engine.py`.

## Module Design

**Exports:** The top-level package exports only the public user API in `nanovllm/__init__.py`. Internal modules import each other directly by absolute package path.

**Barrel Files:** A minimal barrel exists at `nanovllm/__init__.py`. There are no directory-level `__init__.py` barrel exports in `nanovllm/engine/`, `nanovllm/layers/`, `nanovllm/models/`, or `nanovllm/utils/`.

**State Management:** Use explicit object state for engine components (`nanovllm/engine/llm_engine.py`, `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`) and a single dataclass-backed module context for attention execution state (`nanovllm/utils/context.py`).

---

*Convention analysis: 2026-05-11*
