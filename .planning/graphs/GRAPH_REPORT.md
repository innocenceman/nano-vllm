# Graph Report - nano-vllm  (2026-05-12)

## Corpus Check
- 24 files · ~19,017 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 183 nodes · 312 edges · 13 communities (11 shown, 2 thin omitted)
- Extraction: 71% EXTRACTED · 29% INFERRED · 0% AMBIGUOUS · INFERRED: 91 edges (avg confidence: 0.61)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `166d9df1`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- [[_COMMUNITY_Community 0|Community 0]]
- [[_COMMUNITY_Community 1|Community 1]]
- [[_COMMUNITY_Community 2|Community 2]]
- [[_COMMUNITY_Community 3|Community 3]]
- [[_COMMUNITY_Community 4|Community 4]]
- [[_COMMUNITY_Community 5|Community 5]]
- [[_COMMUNITY_Community 6|Community 6]]
- [[_COMMUNITY_Community 7|Community 7]]
- [[_COMMUNITY_Community 8|Community 8]]
- [[_COMMUNITY_Community 9|Community 9]]
- [[_COMMUNITY_Community 10|Community 10]]
- [[_COMMUNITY_Community 11|Community 11]]

## God Nodes (most connected - your core abstractions)
1. `ModelRunner` - 20 edges
2. `Sequence` - 16 edges
3. `Qwen3ForCausalLM` - 14 edges
4. `BlockManager` - 13 edges
5. `LLMEngine` - 13 edges
6. `Scheduler` - 13 edges
7. `RowParallelLinear` - 12 edges
8. `Qwen3Attention` - 12 edges
9. `Qwen3MLP` - 12 edges
10. `Qwen3DecoderLayer` - 12 edges

## Surprising Connections (you probably didn't know these)
- `main()` --calls--> `LLM`  [INFERRED]
  example.py → nanovllm/llm.py
- `main()` --calls--> `SamplingParams`  [INFERRED]
  example.py → nanovllm/sampling_params.py
- `main()` --calls--> `LLM`  [INFERRED]
  bench.py → nanovllm/llm.py
- `main()` --calls--> `SamplingParams`  [INFERRED]
  bench.py → nanovllm/sampling_params.py
- `Sequence` --uses--> `SamplingParams`  [INFERRED]
  nanovllm/engine/sequence.py → nanovllm/sampling_params.py

## Communities (13 total, 2 thin omitted)

### Community 0 - "Community 0"
Cohesion: 0.13
Nodes (13): SiluAndMul, Attention, ParallelLMHead, VocabParallelEmbedding, RMSNorm, MergedColumnParallelLinear, QKVParallelLinear, RowParallelLinear (+5 more)

### Community 1 - "Community 1"
Cohesion: 0.12
Nodes (9): capture_cudagraph(), ModelRunner, run_model(), Sampler, Context, get_context(), reset_context(), set_context() (+1 more)

### Community 2 - "Community 2"
Cohesion: 0.1
Nodes (4): LLMEngine, Scheduler, Sequence, Config

### Community 3 - "Community 3"
Cohesion: 0.12
Nodes (7): SequenceStatus, Enum, LLMEngine, main(), main(), LLM, SamplingParams

### Community 4 - "Community 4"
Cohesion: 0.18
Nodes (4): ColumnParallelLinear, divide(), LinearBase, ReplicatedLinear

### Community 5 - "Community 5"
Cohesion: 0.23
Nodes (3): Block, BlockManager, compute_hash()

### Community 6 - "Community 6"
Cohesion: 0.18
Nodes (10): Benchmark, code:bash (pip install git+https://github.com/GeeeekExplorer/nano-vllm.), code:bash (huggingface-cli download --resume-download Qwen/Qwen3-0.6B \), code:python (from nanovllm import LLM, SamplingParams), Installation, Key Features, Model Download, Nano-vLLM (+2 more)

### Community 7 - "Community 7"
Cohesion: 0.47
Nodes (4): apply_rotary_emb(), forward(), get_rope(), RotaryEmbedding

### Community 8 - "Community 8"
Cohesion: 0.33
Nodes (5): Always Do, CLI, GitNexus — Code Intelligence, Never Do, Resources

### Community 9 - "Community 9"
Cohesion: 0.33
Nodes (5): Always Do, CLI, GitNexus — Code Intelligence, Never Do, Resources

## Knowledge Gaps
- **14 isolated node(s):** `Key Features`, `code:bash (pip install git+https://github.com/GeeeekExplorer/nano-vllm.)`, `code:bash (huggingface-cli download --resume-download Qwen/Qwen3-0.6B \)`, `code:python (from nanovllm import LLM, SamplingParams)`, `Benchmark` (+9 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **2 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `ModelRunner` connect `Community 1` to `Community 0`, `Community 2`?**
  _High betweenness centrality (0.422) - this node is a cross-community bridge._
- **Why does `Qwen3ForCausalLM` connect `Community 0` to `Community 1`?**
  _High betweenness centrality (0.378) - this node is a cross-community bridge._
- **Why does `Sequence` connect `Community 2` to `Community 1`, `Community 3`, `Community 5`?**
  _High betweenness centrality (0.278) - this node is a cross-community bridge._
- **Are the 6 inferred relationships involving `ModelRunner` (e.g. with `Config` and `Sequence`) actually correct?**
  _`ModelRunner` has 6 INFERRED edges - model-reasoned connections that need verification._
- **Are the 8 inferred relationships involving `Sequence` (e.g. with `SamplingParams` and `ModelRunner`) actually correct?**
  _`Sequence` has 8 INFERRED edges - model-reasoned connections that need verification._
- **Are the 10 inferred relationships involving `Qwen3ForCausalLM` (e.g. with `ModelRunner` and `SiluAndMul`) actually correct?**
  _`Qwen3ForCausalLM` has 10 INFERRED edges - model-reasoned connections that need verification._
- **Are the 3 inferred relationships involving `BlockManager` (e.g. with `Sequence` and `Scheduler`) actually correct?**
  _`BlockManager` has 3 INFERRED edges - model-reasoned connections that need verification._