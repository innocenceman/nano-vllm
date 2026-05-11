# Testing Patterns

**Analysis Date:** 2026-05-11

## Test Framework

**Runner:**
- Not detected.
- No `pytest`, `unittest`, `nose`, or test runner configuration appears in `pyproject.toml`.
- No `pytest.ini`, `tox.ini`, `setup.cfg`, `.coveragerc`, `noxfile.py`, or `Makefile` test target is present.

**Assertion Library:**
- Not detected for tests.
- Production code uses Python `assert` statements for invariants in `nanovllm/config.py`, `nanovllm/sampling_params.py`, `nanovllm/layers/attention.py`, `nanovllm/layers/linear.py`, `nanovllm/engine/block_manager.py`, and `nanovllm/models/qwen3.py`.

**Run Commands:**
```bash
# Not detected: no repository test command is defined in `pyproject.toml`, `README.md`, or config files.
# Manual smoke script: `python3 example.py`
# Manual benchmark script: `python3 bench.py`
```

## Test File Organization

**Location:**
- Not detected.
- No `tests/`, `test/`, or co-located `test_*.py`, `*_test.py`, `*.test.py`, or `*.spec.py` files are present.

**Naming:**
- Not detected for tests.
- Runtime modules use package-path names such as `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`, and `nanovllm/models/qwen3.py`.

**Structure:**
```
Not detected
```

## Test Structure

**Suite Organization:**
```python
# Not detected: no test suites, test classes, fixtures, or test functions are present.
```

**Patterns:**
- Setup pattern: not detected.
- Teardown pattern: not detected.
- Assertion pattern: not detected in tests.
- Manual execution pattern: `example.py` defines `main()` and uses `if __name__ == "__main__": main()`.
- Benchmark execution pattern: `bench.py` defines `main()`, seeds random input generation with `seed(0)`, and uses `if __name__ == "__main__": main()`.

## Mocking

**Framework:** Not detected.

**Patterns:**
```python
# Not detected: no `unittest.mock`, `pytest.monkeypatch`, monkeypatch fixtures,
# fake model runners, fake tokenizers, or test doubles are present.
```

**What to Mock:**
- Not established by repository tests.
- External-heavy seams identifiable from source are model/tokenizer loading in `nanovllm/config.py` and `nanovllm/engine/llm_engine.py`, CUDA/distributed setup in `nanovllm/engine/model_runner.py`, and safetensors loading in `nanovllm/utils/loader.py`.

**What NOT to Mock:**
- Not established by repository tests.
- Core tensor math and scheduler/block-manager behavior have no repository test precedent.

## Fixtures and Factories

**Test Data:**
```python
# Not detected: no fixture factories or shared test data modules are present.
```

**Location:**
- Not detected.
- Manual scripts use hard-coded local model paths: `example.py` and `bench.py` expand `~/huggingface/Qwen3-0.6B/`.
- Benchmark input data is generated inline in `bench.py` with seeded random token IDs.

## Coverage

**Requirements:** None enforced.

**View Coverage:**
```bash
# Not detected: no coverage command or coverage configuration is present.
```

- `.gitignore` ignores common Python test and coverage artifacts such as `.pytest_cache/`, `.coverage`, `coverage.xml`, `htmlcov/`, and `cover/`.
- No coverage threshold is configured in `pyproject.toml` or standalone coverage files.

## Test Types

**Unit Tests:**
- Not used.
- Candidate unit-testable modules include `nanovllm/engine/block_manager.py`, `nanovllm/engine/sequence.py`, `nanovllm/sampling_params.py`, and `nanovllm/utils/context.py`, but no tests are present.

**Integration Tests:**
- Not used.
- Integration seams include `LLMEngine` in `nanovllm/engine/llm_engine.py`, `ModelRunner` in `nanovllm/engine/model_runner.py`, Hugging Face configuration/tokenizer loading in `nanovllm/config.py` and `nanovllm/engine/llm_engine.py`, and CUDA/distributed initialization in `nanovllm/engine/model_runner.py`.

**E2E Tests:**
- Not used.
- `example.py` is a manual smoke path for local model generation.
- `bench.py` is a manual performance benchmark path for local model generation.

## Common Patterns

**Async Testing:**
```python
# Not detected: no async tests or async runtime code paths are present.
```

**Error Testing:**
```python
# Not detected: production invariants use `assert`, but no tests assert failure cases.
```

**Manual Smoke Pattern:**
```python
# `example.py` pattern:
# 1. Expand local model path.
# 2. Build tokenizer and `LLM`.
# 3. Build `SamplingParams`.
# 4. Generate outputs for prompts.
# 5. Print prompt/completion pairs.
```

**Manual Benchmark Pattern:**
```python
# `bench.py` pattern:
# 1. `seed(0)` for reproducible random prompt lengths and token IDs.
# 2. Build `LLM` against a local model path.
# 3. Warm up with a single generation call.
# 4. Time `llm.generate(..., use_tqdm=False)`.
# 5. Print total tokens, elapsed time, and throughput.
```

---

*Testing analysis: 2026-05-11*
