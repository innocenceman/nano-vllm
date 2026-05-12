# Change-to-Test Ladder — Scheduler Policy

            **Current label:** `deep-partial`

            ## Evidence Already Collected

            - compileall passed
- GitNexus context found scheduler methods and `LLMEngine` import
- code-review-graph updated but no behavioral tests exist

            ## Minimum Checks for Any Change

            1. Re-run `python3 -m compileall -q nanovllm example.py bench.py`.
            2. Re-run GitNexus analysis or confirm `npx gitnexus status` is up to date.
            3. Run source-specific unit/integration tests from the list below when available.
            4. If no tests exist, add the smallest regression test before behavior changes.

            ## Tests / Checks Needed to Promote

            - CPU unit tests with fake `BlockManager` capacity.
- Regression tests for chunked prefill and preemption ordering.

            ## Exception Handling

            - Do not promote to `verified` unless the relevant check passes in the current environment.
            - If dependencies, GPU, model files, or distributed fixtures are unavailable, keep the label `exceptioned-deep-partial` and record the exact blocker.
