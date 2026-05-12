# Sequence State Model

            **Generated:** 2026-05-12  
            **Coverage label:** `deep-partial`  
            **Logical path:** `engine-runtime/state/sequence-state`

            ## Responsibility

            Represent one request, token windows, prompt/completion split, block table, status, and pickle-friendly child-process state.

            ## Source Paths

            - `nanovllm/engine/sequence.py`
- `nanovllm/sampling_params.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/engine/sequence.py:Sequence`
- `Class:nanovllm/engine/sequence.py:SequenceStatus`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Represent one request, token windows, prompt/completion split, block table, status, and pickle-friendly child-process state.
            - **Bounded code paths:** `nanovllm/engine/sequence.py`, `nanovllm/sampling_params.py`
            - **Owner module:** `engine-runtime/state`
            - **Risk surface:** Empty token list raises an implicit `IndexError` at `last_token`.; Default argument constructs one `SamplingParams()` at import time.; Pickle state discards full token history during decode.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** no automated unit tests exist

            ## Risks / Exceptions

            - Empty token list raises an implicit `IndexError` at `last_token`.
- Default argument constructs one `SamplingParams()` at import time.
- Pickle state discards full token history during decode.
