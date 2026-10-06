# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 7981.6 | 7981.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 5479.8 | 5480.0 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 10174.0 | 10174.2 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **7878.5** · total **7878.6**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the provided context, **Goodput** is more useful than raw throughput because it focuses only on requests per second that met the Target Time-to-Failure (TTFT) and Target Time-to-Poll (TPOT) targets.

In contrast, **Raw Throughput** ignores SLOs (Service Level Objectives). Goodput explicitly states that "Throughput at saturation ignores SLOs," meaning it calculates total throughput without

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** by storing the Key-Value (KV) cache in non-contiguous pages. This design allows the system to remove the wasted internal fragmentation that typically occurs when GPU memory pages are tightly packed or when the KV cache is stored contiguously, thereby optimizing memory usage for compute-bound operations.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps primarily when **compute-bound operations require separate execution paths** or when **memory bandwidth is the limiting factor**.

Based on the provided context:
1.  **Prefill is compute-bound**: It requires significant processing power. Splitting it allows the engine to process compute-intensive tasks on different parts of the pipeline, potentially improving thr


## Which N16-N19 pieces are real

- **N16 Cloud/IaC:** stubbed; this run used the local Windows laptop.
- **N17 Data pipeline:** stubbed; the six documents are static `TOY_DOCS`.
- **N18 Lakehouse:** stubbed; there is no external table or lakehouse store.
- **N19 Vector + features:** stubbed; retrieval is keyword overlap, not a vector index.
- **N20 Serving:** real; all three queries called the local `llama-server` endpoint.

The LLM owning effectively 100% of the 7,878.6 ms mean latency was expected because
the stub embed/retrieve stages cost only 0.1 ms. To halve end-to-end latency I would
attack LLM decode first: test Vulkan offload and a tighter output-token budget, then
remeasure; optimizing the 0.1 ms retrieval path cannot materially move the total.
