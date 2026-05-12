# Change-to-Test Ladder — Activation, RMSNorm, and Rotary Embedding

            **Current label:** `exceptioned-deep-partial`

            ## Evidence Already Collected

            - compileall passed
- source inventory found math primitive files
- runtime blocked by missing `torch`

            ## Minimum Checks for Any Change

            1. Re-run `python3 -m compileall -q nanovllm example.py bench.py`.
            2. Re-run GitNexus analysis or confirm `npx gitnexus status` is up to date.
            3. Run source-specific unit/integration tests from the list below when available.
            4. If no tests exist, add the smallest regression test before behavior changes.

            ## Tests / Checks Needed to Promote

            - CPU/GPU numeric tests against simple reference implementations.
- Cache identity test for `get_rope` LRU behavior.

            ## Exception Handling

            - Do not promote to `verified` unless the relevant check passes in the current environment.
            - If dependencies, GPU, model files, or distributed fixtures are unavailable, keep the label `exceptioned-deep-partial` and record the exact blocker.
