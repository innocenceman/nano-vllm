# Batch Preparation and Forward Context

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `gpu-execution/batching/batch-preparation-context`

            ## Responsibility

            Prepare prefill/decode tensors, slot mappings, block tables, temperatures, and process-local context consumed by attention/LM-head layers.

            ## Source Paths

            - `nanovllm/engine/model_runner.py`
- `nanovllm/utils/context.py`
- `nanovllm/layers/embed_head.py`
- `nanovllm/layers/attention.py`

            ## Candidate Impact Anchors

            - `Method:nanovllm/engine/model_runner.py:ModelRunner.prepare_prefill#1`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.prepare_decode#1`
- `Function:nanovllm/utils/context.py:set_context`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Prepare prefill/decode tensors, slot mappings, block tables, temperatures, and process-local context consumed by attention/LM-head layers.
            - **Bounded code paths:** `nanovllm/engine/model_runner.py`, `nanovllm/utils/context.py`, `nanovllm/layers/embed_head.py`, `nanovllm/layers/attention.py`
            - **Owner module:** `gpu-execution/batching`
            - **Risk surface:** Global `_CONTEXT` is process-local mutable state and must be reset after every run.; Tensor creation pins memory and immediately moves to CUDA, blocking CPU unit tests without seams.; Slot mapping logic is dense and currently untested.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** source-load of context blocked by missing `torch`

            ## Risks / Exceptions

            - Global `_CONTEXT` is process-local mutable state and must be reset after every run.
- Tensor creation pins memory and immediately moves to CUDA, blocking CPU unit tests without seams.
- Slot mapping logic is dense and currently untested.
