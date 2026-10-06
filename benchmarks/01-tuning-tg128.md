# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **14 physical · 20 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 3.5 | 7% |
| 7 | 50.5 | 100% |
| 14 | 49.1 | 97% |
| 20 | 36.7 | 73% |
| 40 | 18.2 | 36% |

**Best**: `-t 7` at 50.5 tok/s
**Slowest tested**: `-t 1` at 3.5 tok/s (14.51x spread)
**Against the physical-core default** (`-t 14`, 49.1 tok/s): 1.03x

Use this in your run:

```bash
LAB_N_THREADS=7 make bench
```

## My explanation

The knee is between 7 and 14 threads: 7 threads reached 50.5 tok/s and 14 was
already 3% slower at 49.1 tok/s. Decode repeatedly streams model weights, so seven
workers appear sufficient to saturate useful memory bandwidth/cache capacity on
this hybrid-core CPU. More workers do not add proportional bandwidth; they add
cache pressure, scheduling and synchronization overhead. That is why 20 threads
dropped to 36.7 tok/s and deliberate oversubscription at 40 fell to 18.2 tok/s.
