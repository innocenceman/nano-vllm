# Code Paths — Safetensors Weight Loading and Packed Mapping

            ## Included Paths

            - `nanovllm/utils/loader.py`
- `nanovllm/models/qwen3.py`
- `nanovllm/layers/linear.py`
- `nanovllm/layers/embed_head.py`

            ## Impact Anchors

            - `Function:nanovllm/utils/loader.py:load_model`
- `Class:nanovllm/models/qwen3.py:Qwen3ForCausalLM`
- `Method:nanovllm/layers/linear.py:QKVParallelLinear.weight_loader#3`

            ## Upstream / Downstream Notes

            - Upstream callers generally enter through `nanovllm/__init__.py`, `nanovllm/llm.py`, `example.py`, or `bench.py` unless this leaf is lower-level model/layer infrastructure.
            - Downstream dependencies follow the generation pipeline documented in `.planning/architecture/ARCHITECTURE-ATLAS.md`.
            - Use GitNexus `context` or `impact` on the anchors above before changing public signatures, scheduling behavior, model runner methods, or tensor-parallel layer constructors.

            ## Exclusions

            - Generated caches such as `__pycache__/`, `.gitnexus/`, `.omx/`, and `.code-review-graph/` are not part of this leaf.
            - GSD control-plane files remain outside this handoff skill's mutation authority.
