# Change-to-Test Ladder — Config Normalization

            **Current label:** `exceptioned-deep-partial`

            ## Evidence Already Collected

            - compileall passed
- GitNexus found `Config` and `__post_init__`
- runtime construction blocked by missing `transformers` and model fixture

            ## Minimum Checks for Any Change

            1. Re-run `python3 -m compileall -q nanovllm example.py bench.py`.
            2. Re-run GitNexus analysis or confirm `npx gitnexus status` is up to date.
            3. Run source-specific unit/integration tests from the list below when available.
            4. If no tests exist, add the smallest regression test before behavior changes.

            ## Tests / Checks Needed to Promote

            - Unit-test invalid model path, invalid tensor-parallel size, invalid KV block size.
- Use a tiny local Hugging Face config fixture to avoid downloading model weights.

            ## Exception Handling

            - Do not promote to `verified` unless the relevant check passes in the current environment.
            - If dependencies, GPU, model files, or distributed fixtures are unavailable, keep the label `exceptioned-deep-partial` and record the exact blocker.
