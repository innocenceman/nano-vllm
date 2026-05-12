# Scheduler Policy

            **Generated:** 2026-05-12  
            **Coverage label:** `deep-partial`  
            **Logical path:** `engine-runtime/scheduling/scheduler-policy`

            ## Responsibility

            Admit waiting sequences, choose prefill versus decode batches, enforce token/sequence budgets, preempt cache-starved decode work, and postprocess completions.

            ## Source Paths

            - `nanovllm/engine/scheduler.py`
- `nanovllm/engine/block_manager.py`
- `nanovllm/engine/sequence.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/engine/scheduler.py:Scheduler`
- `Method:nanovllm/engine/scheduler.py:Scheduler.schedule#0`
- `Method:nanovllm/engine/scheduler.py:Scheduler.postprocess#3`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Admit waiting sequences, choose prefill versus decode batches, enforce token/sequence budgets, preempt cache-starved decode work, and postprocess completions.
            - **Bounded code paths:** `nanovllm/engine/scheduler.py`, `nanovllm/engine/block_manager.py`, `nanovllm/engine/sequence.py`
            - **Owner module:** `engine-runtime/scheduling`
            - **Risk surface:** Decode branch asserts `scheduled_seqs` rather than returning an idle state.; No tests cover chunked prefill, preemption, or EOS/max token finish.; Relies on mutable `Sequence` fields shared with runner.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** code-review-graph updated but no behavioral tests exist

            ## Risks / Exceptions

            - Decode branch asserts `scheduled_seqs` rather than returning an idle state.
- No tests cover chunked prefill, preemption, or EOS/max token finish.
- Relies on mutable `Sequence` fields shared with runner.
