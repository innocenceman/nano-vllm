# External Integrations

**Analysis Date:** 2026-05-11

## APIs & External Services

**Model Ecosystem:**
- Hugging Face model format - Local model config, tokenizer, and weights are loaded from a model directory.
  - SDK/Client: `transformers` through `AutoConfig` in `nanovllm/config.py`, `AutoTokenizer` in `nanovllm/engine/llm_engine.py`, and `Qwen3Config` in `nanovllm/models/qwen3.py`.
  - Auth: Not detected in application code; model path is local and `Config.__post_init__` asserts the path is a directory in `nanovllm/config.py`.
- Hugging Face Hub CLI - Manual model acquisition is documented in `README.md` with `huggingface-cli download` for `Qwen/Qwen3-0.6B`.
  - SDK/Client: external CLI documented only in `README.md`; no `huggingface_hub` import detected in `nanovllm/`.
  - Auth: Not detected; no token or environment variable is referenced by repository code.

**Source Distribution:**
- GitHub repository install - Package installation is documented from `https://github.com/GeeeekExplorer/nano-vllm.git` in `README.md` and homepage metadata is declared in `pyproject.toml`.
  - SDK/Client: `pip` with a Git URL.
  - Auth: Not detected.

**Documentation Badges:**
- Trendshift and Star History badges - External image/link badges are present in `README.md`.
  - SDK/Client: Markdown links only.
  - Auth: Not applicable.

## Data Storage

**Databases:**
- Not detected - No SQL, NoSQL, vector database, ORM, or database client imports were found in `nanovllm/`.
  - Connection: Not applicable.
  - Client: Not applicable.

**File Storage:**
- Local filesystem only - Model directory paths are supplied to `LLM`/`Config` in `nanovllm/engine/llm_engine.py` and validated in `nanovllm/config.py`.
- Local `.safetensors` weights - `nanovllm/utils/loader.py` scans `*.safetensors` files with `glob(os.path.join(path, "*.safetensors"))` and loads tensors through `safetensors.safe_open`.
- Example model storage path - `example.py` and `bench.py` expand `~/huggingface/Qwen3-0.6B/`.

**Caching:**
- In-memory/GPU KV cache - `nanovllm/engine/model_runner.py` allocates KV cache tensors on CUDA based on `gpu_memory_utilization`.
- Prefix cache - `nanovllm/engine/block_manager.py` hashes token blocks with `xxhash` and stores cache metadata in process memory.
- CUDA graph cache - `nanovllm/engine/model_runner.py` captures reusable decode graphs when `enforce_eager=False`.
- External cache service: None detected.

## Authentication & Identity

**Auth Provider:**
- None detected.
  - Implementation: This is a local Python inference library with no HTTP server, user identity layer, session management, or authorization middleware in `nanovllm/`.

## Monitoring & Observability

**Error Tracking:**
- None detected - No Sentry, OpenTelemetry, Prometheus, MLflow, Weights & Biases, or logging SDK imports were found.

**Logs:**
- Progress display uses `tqdm.auto.tqdm` in `nanovllm/engine/llm_engine.py`.
- Benchmark output uses `print` in `bench.py`.
- Example output uses `print` in `example.py`.
- No structured logging configuration was detected.

## CI/CD & Deployment

**Hosting:**
- Source hosting is GitHub, referenced by `pyproject.toml` and `README.md`.
- Runtime hosting/deployment target is not configured; no Dockerfile, compose file, Procfile, server entrypoint, or platform config was detected.

**CI Pipeline:**
- None detected - No `.github/`, `.gitlab/`, or CI workflow configuration files were detected.

## Environment Configuration

**Required env vars:**
- None detected in application code.
- No `.env` or `.env.*` files were present in the repository scan.
- Runtime configuration is supplied through `Config` fields in `nanovllm/config.py` and `LLM` keyword arguments in `nanovllm/engine/llm_engine.py`.

**Secrets location:**
- Not detected.
- `.gitignore` excludes `.env` and `.pypirc`; these are secret/config patterns and should not be committed.
- Hugging Face authentication, if needed for private model download, is outside repository code and should be handled by the user's local Hugging Face CLI configuration.

## Webhooks & Callbacks

**Incoming:**
- None detected - The repository has no HTTP server, API route, webhook handler, or callback endpoint.

**Outgoing:**
- No runtime webhooks or application HTTP callbacks detected.
- Model download is an operator-run external CLI step documented in `README.md`, not an application callback.
- Transformers loading in `nanovllm/config.py` and `nanovllm/engine/llm_engine.py` is local-path oriented because `Config.__post_init__` requires `self.model` to be an existing directory.

---

*Integration audit: 2026-05-11*
