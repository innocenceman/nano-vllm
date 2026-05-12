# File Roles — CUDA Graph Decode Execution

- `nanovllm/engine/model_runner.py` — source path participating in cuda graph decode execution; inspect before changes and update this packet if ownership shifts.

## Ownership Notes

- This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
- If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
