# Code Paths — Batch Preparation and Forward Context

            ## Included Paths

            - `nanovllm/engine/model_runner.py`
- `nanovllm/utils/context.py`
- `nanovllm/layers/embed_head.py`
- `nanovllm/layers/attention.py`

            ## Impact Anchors

            - `Method:nanovllm/engine/model_runner.py:ModelRunner.prepare_prefill#1`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.prepare_decode#1`
- `Function:nanovllm/utils/context.py:set_context`

            ## Upstream / Downstream Notes

            - Upstream callers generally enter through `nanovllm/__init__.py`, `nanovllm/llm.py`, `example.py`, or `bench.py` unless this leaf is lower-level model/layer infrastructure.
            - Downstream dependencies follow the generation pipeline documented in `.planning/architecture/ARCHITECTURE-ATLAS.md`.
            - Use GitNexus `context` or `impact` on the anchors above before changing public signatures, scheduling behavior, model runner methods, or tensor-parallel layer constructors.

            ## Exclusions

            - Generated caches such as `__pycache__/`, `.gitnexus/`, `.omx/`, and `.code-review-graph/` are not part of this leaf.
            - GSD control-plane files remain outside this handoff skill's mutation authority.
