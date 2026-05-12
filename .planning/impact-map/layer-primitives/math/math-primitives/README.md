# Activation, RMSNorm, and Rotary Embedding

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `layer-primitives/math/math-primitives`

            ## Responsibility

            Provide compiled SiLU-gate activation, RMSNorm residual variants, RoPE cache creation, and rotary application.

            ## Source Paths

            - `nanovllm/layers/activation.py`
- `nanovllm/layers/layernorm.py`
- `nanovllm/layers/rotary_embedding.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/layers/activation.py:SiluAndMul`
- `Class:nanovllm/layers/layernorm.py:RMSNorm`
- `Function:nanovllm/layers/rotary_embedding.py:get_rope`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Provide compiled SiLU-gate activation, RMSNorm residual variants, RoPE cache creation, and rotary application.
            - **Bounded code paths:** `nanovllm/layers/activation.py`, `nanovllm/layers/layernorm.py`, `nanovllm/layers/rotary_embedding.py`
            - **Owner module:** `layer-primitives/math`
            - **Risk surface:** `torch.compile` behavior is untested in target runtime.; RoPE requires rotary dim equal head size via assert.; No numeric reference tests.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`

            ## Risks / Exceptions

            - `torch.compile` behavior is untested in target runtime.
- RoPE requires rotary dim equal head size via assert.
- No numeric reference tests.
