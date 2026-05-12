# Vocab-Parallel Embedding and LM Head

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `layer-primitives/tensor-parallel/embedding-lm-head`

            ## Responsibility

            Shard vocab embeddings and gather LM-head logits across tensor-parallel ranks while selecting final prefill logits via context.

            ## Source Paths

            - `nanovllm/layers/embed_head.py`
- `nanovllm/utils/context.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/layers/embed_head.py:VocabParallelEmbedding`
- `Class:nanovllm/layers/embed_head.py:ParallelLMHead`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Shard vocab embeddings and gather LM-head logits across tensor-parallel ranks while selecting final prefill logits via context.
            - **Bounded code paths:** `nanovllm/layers/embed_head.py`, `nanovllm/utils/context.py`
            - **Owner module:** `layer-primitives/tensor-parallel`
            - **Risk surface:** Requires initialized distributed state at construction.; Rank-0 gather behavior and mask math are untested.; Depends on context shape for prefill logits selection.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime blocked by missing `torch`

            ## Risks / Exceptions

            - Requires initialized distributed state at construction.
- Rank-0 gather behavior and mask math are untested.
- Depends on context shape for prefill logits selection.
