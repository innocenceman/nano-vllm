# Code Paths — Qwen3 Model Composition

            ## Included Paths

            - `nanovllm/models/qwen3.py`
- `nanovllm/layers/*.py`

            ## Impact Anchors

            - `Class:nanovllm/models/qwen3.py:Qwen3ForCausalLM`
- `Class:nanovllm/models/qwen3.py:Qwen3Model`
- `Class:nanovllm/models/qwen3.py:Qwen3DecoderLayer`

            ## Upstream / Downstream Notes

            - Upstream callers generally enter through `nanovllm/__init__.py`, `nanovllm/llm.py`, `example.py`, or `bench.py` unless this leaf is lower-level model/layer infrastructure.
            - Downstream dependencies follow the generation pipeline documented in `.planning/architecture/ARCHITECTURE-ATLAS.md`.
            - Use GitNexus `context` or `impact` on the anchors above before changing public signatures, scheduling behavior, model runner methods, or tensor-parallel layer constructors.

            ## Exclusions

            - Generated caches such as `__pycache__/`, `.gitnexus/`, `.omx/`, and `.code-review-graph/` are not part of this leaf.
            - GSD control-plane files remain outside this handoff skill's mutation authority.
