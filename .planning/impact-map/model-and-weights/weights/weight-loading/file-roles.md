# File Roles — Safetensors Weight Loading and Packed Mapping

            - `nanovllm/utils/loader.py` — source path participating in safetensors weight loading and packed mapping; inspect before changes and update this packet if ownership shifts.
- `nanovllm/models/qwen3.py` — source path participating in safetensors weight loading and packed mapping; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/linear.py` — source path participating in safetensors weight loading and packed mapping; inspect before changes and update this packet if ownership shifts.
- `nanovllm/layers/embed_head.py` — source path participating in safetensors weight loading and packed mapping; inspect before changes and update this packet if ownership shifts.

            ## Ownership Notes

            - This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
            - If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
