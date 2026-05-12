# File Roles — Sequence State Model

            - `nanovllm/engine/sequence.py` — source path participating in sequence state model; inspect before changes and update this packet if ownership shifts.
- `nanovllm/sampling_params.py` — source path participating in sequence state model; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
