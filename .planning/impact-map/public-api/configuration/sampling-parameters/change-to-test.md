# Change-to-Test Ladder — Sampling Parameters

            **Current label:** `verified`

            ## Evidence Already Collected

            - compileall passed
- direct source-load of `nanovllm/sampling_params.py` passed
- source inspection confirmed dataclass-only dependency surface

            ## Minimum Checks for Any Change

            1. Re-run `python3 -m compileall -q nanovllm example.py bench.py`.
            2. Re-run GitNexus analysis or confirm `npx gitnexus status` is up to date.
            3. Run source-specific unit/integration tests from the list below when available.
            4. If no tests exist, add the smallest regression test before behavior changes.

            ## Tests / Checks Needed to Promote

            - Add CPU unit tests for temperature guard, max token validation, and copying values into `Sequence`.

            ## Exception Handling

            - Do not promote to `verified` unless the relevant check passes in the current environment.
            - If dependencies, GPU, model files, or distributed fixtures are unavailable, keep the label `exceptioned-deep-partial` and record the exact blocker.
