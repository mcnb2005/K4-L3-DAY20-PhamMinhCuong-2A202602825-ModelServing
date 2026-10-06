# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=7` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3330 | 226 / 286 | 45.3 / 49.8 | 3011 / 3363 / 3363 | 22.1 |
| UD-Q2_K_XL | 0.39 | 2791 | 290 / 356 | 36.8 / 44.1 | 2606 / 3078 / 3078 | 27.2 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.23x faster** than `Q4_K_M` here, for 0.11 GB less on disk.

## My observation

`UD-Q2_K_XL` used 0.11 GB less disk and decoded 1.23x faster (27.2 vs 22.1
tok/s), although its TTFT P50 was 28% higher (290 vs 226 ms). With the same
goodput question, both answers kept a clear three-bullet structure, but Q2 drifted
toward monitoring and autoscaling instead of defining goodput directly. I would use
Q2 for throughput-sensitive local serving, and keep Q4 for quality-sensitive answers.
