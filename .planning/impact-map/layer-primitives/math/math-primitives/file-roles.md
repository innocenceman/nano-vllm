# File Roles — Activation, RMSNorm, and Rotary Embedding

            - `nanovllm/layers/activation.py` — source path participating in activation, rmsnorm, and rotary embedding; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/layernorm.py` — source path participating in activation, rmsnorm, and rotary embedding; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/rotary_embedding.py` — source path participating in activation, rmsnorm, and rotary embedding; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
