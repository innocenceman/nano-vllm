# File Roles — Qwen3 Model Composition

            - `nanovllm/models/qwen3.py` — source path participating in qwen3 model composition; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/*.py` — source path participating in qwen3 model composition; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
