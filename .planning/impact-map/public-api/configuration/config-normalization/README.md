# Config Normalization

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `public-api/configuration/config-normalization`

            ## Responsibility

            Normalize engine capacity, tensor-parallel, eager/graph, KV-cache, and Hugging Face model metadata.

            ## Source Paths

            - `nanovllm/config.py`
- `pyproject.toml`

            ## Candidate Impact Anchors

            - `Class:nanovllm/config.py:Config`
- `Method:nanovllm/config.py:Config.__post_init__#0`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Normalize engine capacity, tensor-parallel, eager/graph, KV-cache, and Hugging Face model metadata.
            - **Bounded code paths:** `nanovllm/config.py`, `pyproject.toml`
            - **Owner module:** `public-api/configuration`
            - **Risk surface:** Uses public-facing `assert` validation.; Requires local model directory and `transformers.AutoConfig` at construction time.; Caps `max_model_len` to Hugging Face config without separate observability.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime construction blocked by missing `transformers` and model fixture

            ## Risks / Exceptions

            - Uses public-facing `assert` validation.
- Requires local model directory and `transformers.AutoConfig` at construction time.
- Caps `max_model_len` to Hugging Face config without separate observability.
