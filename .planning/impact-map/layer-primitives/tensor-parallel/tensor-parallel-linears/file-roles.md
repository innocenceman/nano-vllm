# File Roles — Tensor-Parallel Linear Layers

- `nanovllm/layers/linear.py` — source path participating in tensor-parallel linear layers; inspect before changes and update this packet if ownership shifts.

## Ownership Notes

- This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
- If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
