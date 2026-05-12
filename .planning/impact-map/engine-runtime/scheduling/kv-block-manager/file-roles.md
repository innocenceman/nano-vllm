# File Roles — KV Block Manager and Prefix Cache Metadata

- `nanovllm/engine/block_manager.py` — source path participating in kv block manager and prefix cache metadata; inspect before changes and update this packet if ownership shifts.

## Ownership Notes

- This leaf is intentionally narrower than the physical directory when the directory contains mixed responsibilities.
- If a planned phase touches sources outside the included paths, update `MODULE-INDEX.md` and consider splitting or moving the phase boundary.
