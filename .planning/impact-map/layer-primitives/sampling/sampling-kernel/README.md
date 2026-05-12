# Temperature Sampling Kernel

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `layer-primitives/sampling/sampling-kernel`

            ## Responsibility

            Convert logits and per-sequence temperatures into sampled token IDs using exponential-race sampling.

            ## Source Paths

            - `nanovllm/layers/sampler.py`
- `nanovllm/sampling_params.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/layers/sampler.py:Sampler`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.prepare_sample#1`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Convert logits and per-sequence temperatures into sampled token IDs using exponential-race sampling.
            - **Bounded code paths:** `nanovllm/layers/sampler.py`, `nanovllm/sampling_params.py`
            - **Owner module:** `layer-primitives/sampling`
            - **Risk surface:** No deterministic seed or distribution tests.; Greedy mode is disallowed at `SamplingParams`; sampler assumes positive temperatures.; Uses in-place operations on logits/probs.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`

            ## Risks / Exceptions

            - No deterministic seed or distribution tests.
- Greedy mode is disallowed at `SamplingParams`; sampler assumes positive temperatures.
- Uses in-place operations on logits/probs.
