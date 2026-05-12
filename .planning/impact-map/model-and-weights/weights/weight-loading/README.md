# Safetensors Weight Loading and Packed Mapping

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `model-and-weights/weights/weight-loading`

            ## Responsibility

            Load Hugging Face safetensors files, apply packed Q/K/V and gate/up mappings, and dispatch to parameter-specific sharding loaders.

            ## Source Paths

            - `nanovllm/utils/loader.py`
- `nanovllm/models/qwen3.py`
- `nanovllm/layers/linear.py`
- `nanovllm/layers/embed_head.py`

            ## Candidate Impact Anchors

            - `Function:nanovllm/utils/loader.py:load_model`
- `Class:nanovllm/models/qwen3.py:Qwen3ForCausalLM`
- `Method:nanovllm/layers/linear.py:QKVParallelLinear.weight_loader#3`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Load Hugging Face safetensors files, apply packed Q/K/V and gate/up mappings, and dispatch to parameter-specific sharding loaders.
            - **Bounded code paths:** `nanovllm/utils/loader.py`, `nanovllm/models/qwen3.py`, `nanovllm/layers/linear.py`, `nanovllm/layers/embed_head.py`
            - **Owner module:** `model-and-weights/weights`
            - **Risk surface:** Missing or unexpected weight names surface through `get_parameter` exceptions without context.; No tests cover packed-name replacement or tensor-parallel sharding.; Requires `safetensors` and `torch`, both unavailable in validation environment.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`/`safetensors`

            ## Risks / Exceptions

            - Missing or unexpected weight names surface through `get_parameter` exceptions without context.
- No tests cover packed-name replacement or tensor-parallel sharding.
- Requires `safetensors` and `torch`, both unavailable in validation environment.
