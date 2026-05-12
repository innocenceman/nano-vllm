# Example and Benchmark Scripts

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `repository-ops/scripts/manual-scripts`

            ## Responsibility

            Provide manual inference smoke and throughput benchmark entry points using a local Qwen3 model directory.

            ## Source Paths

            - `example.py`
- `bench.py`
- `README.md`

            ## Candidate Impact Anchors

            - `Function:example.py:main`
- `Function:bench.py:main`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Provide manual inference smoke and throughput benchmark entry points using a local Qwen3 model directory.
            - **Bounded code paths:** `example.py`, `bench.py`, `README.md`
            - **Owner module:** `repository-ops/scripts`
            - **Risk surface:** Hard-coded `~/huggingface/Qwen3-0.6B/` path.; Manual scripts require full runtime dependency stack, local model, and GPU.; `bench.py` variable typo `max_ouput_len` is harmless but visible debt.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** manual runtime blocked by missing dependencies/model/GPU

            ## Risks / Exceptions

            - Hard-coded `~/huggingface/Qwen3-0.6B/` path.
- Manual scripts require full runtime dependency stack, local model, and GPU.
- `bench.py` variable typo `max_ouput_len` is harmless but visible debt.
