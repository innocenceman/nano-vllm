# Code Paths — Runtime KV Cache Allocation and Attachment

            ## Included Paths

            - `nanovllm/engine/model_runner.py`
- `nanovllm/layers/attention.py`

            ## Impact Anchors

            - `Method:nanovllm/engine/model_runner.py:ModelRunner.allocate_kv_cache#0`
- `Function:nanovllm/layers/attention.py:store_kvcache`

            ## Upstream / Downstream Notes

            - Upstream callers generally enter through `nanovllm/__init__.py`, `nanovllm/llm.py`, `example.py`, or `bench.py` unless this leaf is lower-level model/layer infrastructure.
            - Downstream dependencies follow the generation pipeline documented in `.planning/architecture/ARCHITECTURE-ATLAS.md`.
            - Use GitNexus `context` or `impact` on the anchors above before changing public signatures, scheduling behavior, model runner methods, or tensor-parallel layer constructors.

            ## Exclusions

            - Generated caches such as `__pycache__/`, `.gitnexus/`, `.omx/`, and `.code-review-graph/` are not part of this leaf.
            - GSD control-plane files remain outside this handoff skill's mutation authority.
