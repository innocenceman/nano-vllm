# Technology Stack

**Analysis Date:** 2026-05-11

## Languages

**Primary:**
- Python `>=3.10,<3.13` - Package source in `nanovllm/`, examples in `example.py`, benchmark harness in `bench.py`; requirement declared in `pyproject.toml`.

**Secondary:**
- Triton kernel language - Inline GPU kernel definitions in `nanovllm/layers/attention.py` via `triton.jit` and `triton.language`.
- Markdown - Project documentation in `README.md` and generated planning docs in `.planning/codebase/`.

## Runtime

**Environment:**
- CPython `>=3.10,<3.13` as declared by `pyproject.toml`.
- CUDA-capable PyTorch runtime - The engine sets CUDA as the default device, allocates GPU KV cache, uses CUDA graphs, and synchronizes CUDA in `nanovllm/engine/model_runner.py`.
- NVIDIA NCCL distributed runtime - Tensor parallel workers initialize `torch.distributed` with `nccl` and `tcp://localhost:2333` in `nanovllm/engine/model_runner.py`.

**Package Manager:**
- pip / PEP 517 build backend through `pyproject.toml`.
- Build backend: `setuptools.build_meta` with `setuptools>=61` in `pyproject.toml`.
- Lockfile: missing; no `uv.lock`, `poetry.lock`, `Pipfile.lock`, or `requirements.txt` detected.

## Frameworks

**Core:**
- PyTorch `>=2.4.0` - Model layers, tensor operations, distributed collectives, CUDA memory management, `torch.compile`, inference mode, and CUDA graphs across `nanovllm/engine/model_runner.py`, `nanovllm/layers/`, and `nanovllm/models/qwen3.py`.
- Transformers `>=4.51.0` - Hugging Face configuration/tokenizer loading through `AutoConfig` in `nanovllm/config.py`, `AutoTokenizer` in `nanovllm/engine/llm_engine.py`, and `Qwen3Config` in `nanovllm/models/qwen3.py`.
- Triton `>=3.0.0` - Custom KV-cache storage kernel in `nanovllm/layers/attention.py`.
- FlashAttention `flash-attn` - Variable-length prefill attention and decode attention with KV cache in `nanovllm/layers/attention.py`.

**Testing:**
- Not detected - No pytest/unittest configuration or test files are present under the repository root.

**Build/Dev:**
- setuptools `>=61` - Package discovery for `nanovllm*` configured in `pyproject.toml`.
- tqdm - Progress display for generation throughput in `nanovllm/engine/llm_engine.py`; imported directly and satisfied by the Python environment or transitive dependencies.
- safetensors - Local model weight loading in `nanovllm/utils/loader.py`; imported directly and usually supplied by the Transformers ecosystem.
- NumPy - Prefix-cache hash input serialization in `nanovllm/engine/block_manager.py`; imported directly and usually supplied by PyTorch/Transformers environments.

## Key Dependencies

**Critical:**
- `torch>=2.4.0` - Required for all model execution, CUDA, tensor parallelism, compilation, and sampling in `nanovllm/engine/model_runner.py`, `nanovllm/layers/`, and `nanovllm/models/qwen3.py`.
- `triton>=3.0.0` - Required for the custom KV-cache storage kernel in `nanovllm/layers/attention.py`.
- `transformers>=4.51.0` - Required for model config and tokenizer compatibility with Qwen3 in `nanovllm/config.py`, `nanovllm/engine/llm_engine.py`, and `nanovllm/models/qwen3.py`.
- `flash-attn` - Required for FlashAttention prefill and decode paths in `nanovllm/layers/attention.py`.
- `xxhash` - Required for prefix-cache block hashing in `nanovllm/engine/block_manager.py`.

**Infrastructure:**
- `setuptools>=61` - Build system requirement in `pyproject.toml`.
- `safetensors` - Runtime weight-file reader in `nanovllm/utils/loader.py`.
- `tqdm` - Runtime progress bar in `nanovllm/engine/llm_engine.py`.
- `numpy` - Runtime array conversion for cache hashing in `nanovllm/engine/block_manager.py`.
- Python standard library multiprocessing/shared memory - Tensor-parallel worker orchestration in `nanovllm/engine/llm_engine.py` and `nanovllm/engine/model_runner.py`.

## Configuration

**Environment:**
- Primary runtime configuration is constructor/dataclass based: `LLM(model, **kwargs)` in `nanovllm/engine/llm_engine.py` maps keyword arguments to `Config` fields in `nanovllm/config.py`.
- Required model location is a local directory path; `Config.__post_init__` asserts `os.path.isdir(self.model)` in `nanovllm/config.py`.
- Example and benchmark model paths use `~/huggingface/Qwen3-0.6B/` in `example.py` and `bench.py`.
- No required environment variables or `.env` files were detected; `.gitignore` excludes `.env`, virtualenv directories, and `.pypirc`.

**Build:**
- `pyproject.toml` defines package metadata, Python version bounds, dependencies, and setuptools package discovery.
- `README.md` documents GitHub-based pip installation and manual Hugging Face model download.
- No lint, formatting, test, Docker, or CI configuration files were detected.

## Platform Requirements

**Development:**
- Python `>=3.10,<3.13`.
- CUDA-capable NVIDIA GPU with PyTorch CUDA support; `nanovllm/engine/model_runner.py` unconditionally calls CUDA APIs.
- NCCL backend available for tensor parallel execution; `tensor_parallel_size` must be between 1 and 8 in `nanovllm/config.py`.
- Local Hugging Face-format Qwen3 model directory containing `.safetensors` weights for `nanovllm/utils/loader.py`.

**Production:**
- Deployment target is a Python package/library, not a standalone server; public API is exported from `nanovllm/__init__.py`.
- Installation path documented as `pip install git+https://github.com/GeeeekExplorer/nano-vllm.git` in `README.md`.
- Runtime expects local model files and GPU access; no container, managed hosting, or service process configuration was detected.

---

*Stack analysis: 2026-05-11*
