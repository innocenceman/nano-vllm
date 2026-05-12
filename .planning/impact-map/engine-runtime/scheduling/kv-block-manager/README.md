# KV Block Manager and Prefix Cache Metadata

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `engine-runtime/scheduling/kv-block-manager`

            ## Responsibility

            Allocate/deallocate KV block IDs, maintain reference counts, hash full prompt blocks, and expose prefix-cache reuse decisions to scheduler and runner.

            ## Source Paths

            - `nanovllm/engine/block_manager.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/engine/block_manager.py:BlockManager`
- `Method:nanovllm/engine/block_manager.py:BlockManager.can_allocate#1`
- `Method:nanovllm/engine/block_manager.py:BlockManager.hash_blocks#1`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Allocate/deallocate KV block IDs, maintain reference counts, hash full prompt blocks, and expose prefix-cache reuse decisions to scheduler and runner.
            - **Bounded code paths:** `nanovllm/engine/block_manager.py`
            - **Owner module:** `engine-runtime/scheduling`
            - **Risk surface:** Depends on `xxhash` and `numpy`, unavailable in current validation environment.; Reference-count and free-list invariants are assert-only.; Prefix hash collisions are guarded by token comparison but not tested.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime import/unit checks blocked by missing `xxhash`/`numpy`

            ## Risks / Exceptions

            - Depends on `xxhash` and `numpy`, unavailable in current validation environment.
- Reference-count and free-list invariants are assert-only.
- Prefix hash collisions are guarded by token comparison but not tested.
