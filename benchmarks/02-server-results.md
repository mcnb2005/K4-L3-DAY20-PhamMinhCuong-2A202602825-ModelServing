# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=7` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 30 | 0.53 | 16000 | 23000 | 23000 | 8.5 | 0.0% |
| 50 | 42 | 0.72 | 38000 | 55000 | 57000 | 23.5 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.34x** (27% of linear) |
| P95 latency | **2.39x** |
| Effective concurrency at 50 users | 23.5 vs `--parallel 4` slots (occupancy/slot ratio 5.87) |

**Saturated.** Throughput delivered only 1.34x for 5x the offered load, and effective concurrency (23.5) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.34x while P95 moved 2.39x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## My reading

The server is saturated before or by 50 users: offered load rose 5x but delivered
RPS only 1.34x, while P95 rose 2.39x to 55 s. The strongest evidence is 3.91/4 busy
slots together with 46 deferred requests, so most added latency is queue time. I
would first test `--parallel 8` because RAM is ample and the queue is explicit, then
keep it only if a repeated run raises goodput at the chosen P95 SLO without inflating
TPOT enough to cancel the benefit.
