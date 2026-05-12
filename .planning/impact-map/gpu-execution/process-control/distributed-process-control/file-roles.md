# File Roles — Distributed Process Control

            - `nanovllm/engine/model_runner.py` — source path participating in distributed process control; inspect before changes and update this packet if ownership shifts.
- `nanovllm/engine/llm_engine.py` — source path participating in distributed process control; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
