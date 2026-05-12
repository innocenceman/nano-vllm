# Sampling Parameters

            **Generated:** 2026-05-12  
            **Coverage label:** `verified`  
            **Logical path:** `public-api/configuration/sampling-parameters`

            ## Responsibility

            Carry per-request sampling knobs from public API into sequence state and sampler temperature tensors.

            ## Source Paths

            - `nanovllm/sampling_params.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/sampling_params.py:SamplingParams`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Carry per-request sampling knobs from public API into sequence state and sampler temperature tensors.
            - **Bounded code paths:** `nanovllm/sampling_params.py`
            - **Owner module:** `public-api/configuration`
            - **Risk surface:** Greedy sampling is rejected by assertion rather than typed exception.; `max_tokens` and `ignore_eos` have no range/type validation.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** source inspection confirmed dataclass-only dependency surface

            ## Risks / Exceptions

            - Greedy sampling is rejected by assertion rather than typed exception.
- `max_tokens` and `ignore_eos` have no range/type validation.
