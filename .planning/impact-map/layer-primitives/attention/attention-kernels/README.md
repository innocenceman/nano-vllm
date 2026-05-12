# Attention and KV-Cache Kernels

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `layer-primitives/attention/attention-kernels`

            ## Responsibility

            Write keys/values into KV cache with Triton and invoke FlashAttention varlen/paged-cache kernels for prefill and decode.

            ## Source Paths

            - `nanovllm/layers/attention.py`
- `nanovllm/utils/context.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/layers/attention.py:Attention`
- `Function:nanovllm/layers/attention.py:store_kvcache_kernel`
- `Function:nanovllm/layers/attention.py:store_kvcache`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Write keys/values into KV cache with Triton and invoke FlashAttention varlen/paged-cache kernels for prefill and decode.
            - **Bounded code paths:** `nanovllm/layers/attention.py`, `nanovllm/utils/context.py`
            - **Owner module:** `layer-primitives/attention`
            - **Risk surface:** Requires `triton`, `flash_attn`, CUDA tensors, and valid context.; Stride and shape invariants are assert-only.; Prefill prefix-cache branch depends on block-table shape.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`, `triton`, `flash_attn`

            ## Risks / Exceptions

            - Requires `triton`, `flash_attn`, CUDA tensors, and valid context.
- Stride and shape invariants are assert-only.
- Prefill prefix-cache branch depends on block-table shape.
