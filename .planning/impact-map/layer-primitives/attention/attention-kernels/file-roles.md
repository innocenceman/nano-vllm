# File Roles — Attention and KV-Cache Kernels

            - `nanovllm/layers/attention.py` — source path participating in attention and kv-cache kernels; inspect before changes and update this packet if ownership shifts.
- `nanovllm/utils/context.py` — source path participating in attention and kv-cache kernels; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
