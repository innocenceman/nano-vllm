# Distributed Process Control

            **Generated:** 2026-05-12  
            **Coverage label:** `exceptioned-deep-partial`  
            **Logical path:** `gpu-execution/process-control/distributed-process-control`

            ## Responsibility

            Initialize NCCL process groups, select CUDA rank, coordinate tensor-parallel child processes through shared memory, and tear down distributed state.

            ## Source Paths

            - `nanovllm/engine/model_runner.py`
- `nanovllm/engine/llm_engine.py`

            ## Candidate Impact Anchors

            - `Class:nanovllm/engine/model_runner.py:ModelRunner`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.call`
- `Method:nanovllm/engine/model_runner.py:ModelRunner.write_shm`

            ## Phase-Plan Readiness

            - **Primary responsibility:** Initialize NCCL process groups, select CUDA rank, coordinate tensor-parallel child processes through shared memory, and tear down distributed state.
            - **Bounded code paths:** `nanovllm/engine/model_runner.py`, `nanovllm/engine/llm_engine.py`
            - **Owner module:** `gpu-execution/process-control`
            - **Risk surface:** Hard-coded rendezvous `tcp://localhost:2333` and shared-memory name `nanovllm` can collide across concurrent engines.; Shared-memory payload size is fixed at 1 MiB without bounds checks.; No CPU-safe fake distributed test seam.
            - **Minimum validation ladder:** see `change-to-test.md`.
            - **Status reason:** runtime validation blocked by missing `torch` and no CUDA/NCCL fixture

            ## Risks / Exceptions

            - Hard-coded rendezvous `tcp://localhost:2333` and shared-memory name `nanovllm` can collide across concurrent engines.
- Shared-memory payload size is fixed at 1 MiB without bounds checks.
- No CPU-safe fake distributed test seam.
