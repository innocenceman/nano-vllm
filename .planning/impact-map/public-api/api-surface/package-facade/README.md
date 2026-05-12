# Package Facade

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `public-api/api-surface/package-facade`

            ## Responsibility

            Expose the minimal import surface for `LLM` and `SamplingParams` while keeping the public class name stable.

            ## Source Paths

            - `nanovllm/__init__.py`
- `nanovllm/llm.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/llm.py:LLM`
- `Class:nanovllm/engine/llm_engine.py:LLMEngine`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Expose the minimal import surface for `LLM` and `SamplingParams` while keeping the public class name stable.
            - **Bounded code paths:** `nanovllm/__init__.py`, `nanovllm/llm.py`
            - **Owner module:** `public-api/api-surface`
            - **Risk surface:** Public imports currently load the full engine dependency chain.; Import smoke failed in this environment because runtime dependencies, including undeclared `tqdm`, are unavailable.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** package import smoke blocked by missing dependencies

            ## Risks / Exceptions

            - Public imports currently load the full engine dependency chain.
- Import smoke failed in this environment because runtime dependencies, including undeclared `tqdm`, are unavailable.
