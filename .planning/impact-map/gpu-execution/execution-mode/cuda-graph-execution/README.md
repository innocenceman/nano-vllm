# CUDA Graph Decode Execution

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `gpu-execution/execution-mode/cuda-graph-execution`

            ## Responsibility

            Capture decode batch-size CUDA graphs and replay them for non-prefill decode batches when eager mode is disabled.

            ## Source Paths

            - `nanovllm/engine/model_runner.py`

            ## Candidate Impact Anchors

            - `Method:nanovllm/engine/model_runner.py:ModelRunner.capture_cudagraph#0`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.run_model#3`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Capture decode batch-size CUDA graphs and replay them for non-prefill decode batches when eager mode is disabled.
            - **Bounded code paths:** `nanovllm/engine/model_runner.py`
            - **Owner module:** `gpu-execution/execution-mode`
            - **Risk surface:** Capture path assumes valid CUDA graph environment and graph batch-size coverage.; No fallback test for graph replay state mutation.; Deletes graph attributes on exit only when eager is false.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by no CUDA/torch fixture

            ## Risks / Exceptions

            - Capture path assumes valid CUDA graph environment and graph batch-size coverage.
- No fallback test for graph replay state mutation.
- Deletes graph attributes on exit only when eager is false.
