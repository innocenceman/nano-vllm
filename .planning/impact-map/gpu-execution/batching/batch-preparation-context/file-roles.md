# File Roles — Batch Preparation and Forward Context

            - `nanovllm/engine/model_runner.py` — source path participating in batch preparation and forward context; inspect before changes and update this packet if ownership shifts.
- `nanovllm/utils/context.py` — source path participating in batch preparation and forward context; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/embed_head.py` — source path participating in batch preparation and forward context; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/attention.py` — source path participating in batch preparation and forward context; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
