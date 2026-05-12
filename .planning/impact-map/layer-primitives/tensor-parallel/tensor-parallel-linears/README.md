# Tensor-Parallel Linear Layers

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `layer-primitives/tensor-parallel/tensor-parallel-linears`

            ## Responsibility

            Provide replicated, column-parallel, merged-column, QKV, and row-parallel linear layers with sharded weight loaders and distributed reductions.

            ## Source Paths

            - `nanovllm/layers/linear.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/layers/linear.py:QKVParallelLinear`
- `Class:nanovllm/layers/linear.py:RowParallelLinear`
- `Function:nanovllm/layers/linear.py:divide`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Provide replicated, column-parallel, merged-column, QKV, and row-parallel linear layers with sharded weight loaders and distributed reductions.
            - **Bounded code paths:** `nanovllm/layers/linear.py`
            - **Owner module:** `layer-primitives/tensor-parallel`
            - **Risk surface:** Constructors call `dist.get_rank/world_size`, requiring initialized distributed state.; Divisibility assumptions use asserts.; Sharding math has no unit tests.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch` and distributed fixture

            ## Risks / Exceptions

            - Constructors call `dist.get_rank/world_size`, requiring initialized distributed state.
- Divisibility assumptions use asserts.
- Sharding math has no unit tests.
