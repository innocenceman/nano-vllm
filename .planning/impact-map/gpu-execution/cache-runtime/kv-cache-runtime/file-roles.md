# File Roles — Runtime KV Cache Allocation and Attachment

            - `nanovllm/engine/model_runner.py` — source path participating in runtime kv cache allocation and attachment; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/attention.py` — source path participating in runtime kv cache allocation and attachment; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
