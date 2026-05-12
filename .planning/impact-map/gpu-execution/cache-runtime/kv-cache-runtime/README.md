# Runtime KV Cache Allocation and Attachment

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `gpu-execution/cache-runtime/kv-cache-runtime`

            ## Responsibility

            Compute GPU KV-cache capacity from CUDA memory stats, allocate cache tensors, and attach per-layer key/value cache slices to attention modules.

            ## Source Paths

            - `nanovllm/engine/model_runner.py`
- `nanovllm/layers/attention.py`

            ## Candidate Impact Anchors

            - `Method:nanovllm/engine/model_runner.py:ModelRunner.allocate_kv_cache#0`
- `Function:nanovllm/layers/attention.py:store_kvcache`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Compute GPU KV-cache capacity from CUDA memory stats, allocate cache tensors, and attach per-layer key/value cache slices to attention modules.
            - **Bounded code paths:** `nanovllm/engine/model_runner.py`, `nanovllm/layers/attention.py`
            - **Owner module:** `gpu-execution/cache-runtime`
            - **Risk surface:** Requires CUDA memory APIs at engine construction time.; Cache block count uses assert-only positive validation.; Layer cache attachment depends on module traversal order.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`, `triton`, and `flash_attn`

            ## Risks / Exceptions

            - Requires CUDA memory APIs at engine construction time.
- Cache block count uses assert-only positive validation.
- Layer cache attachment depends on module traversal order.
