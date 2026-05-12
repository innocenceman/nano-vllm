# Change-to-Test Ladder — Request Lifecycle and Generation Loop

            **Current label:** `exceptioned-deep-partial`

            ## Evidence Already Collected

            - compileall passed
- GitNexus query surfaced generation flows from scripts through engine methods
- import smoke blocked by missing dependencies

            ## Minimum Checks for Any Change

            1. Re-run `python3 -m compileall -q nanovllm example.py bench.py`.
            2. Re-run GitNexus analysis or confirm `npx gitnexus status` is up to date.
            3. Run source-specific unit/integration tests from the list below when available.
            4. If no tests exist, add the smallest regression test before behavior changes.

            ## Tests / Checks Needed to Promote

            - Import-only test for public package.
- CPU fake-runner test for `generate` loop ordering and output decode.
- Lifecycle test that `exit` is idempotent.

            ## Exception Handling

            - Do not promote to `verified` unless the relevant check passes in the current environment.
            - If dependencies, GPU, model files, or distributed fixtures are unavailable, keep the label `exceptioned-deep-partial` and record the exact blocker.
