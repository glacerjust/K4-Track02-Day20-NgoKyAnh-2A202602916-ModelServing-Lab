# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=12` ·
`ngl=99`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 156 | 2.66 | 2700 | 4200 | 5200 | 7.6 | 0.0% |
| 50 | 139 | 2.39 | 17000 | 22000 | 24000 | 38.2 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **0.90x** (18% of linear) |
| P95 latency | **5.24x** |
| Effective concurrency at 50 users | 38.2 vs `--parallel 4` slots (occupancy/slot ratio 9.54) |

**Saturated.** Throughput delivered only 0.90x for 5x the offered load, and effective concurrency (38.2) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 0.90x while P95 moved 5.24x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Your reading

Server đã hoàn toàn bão hòa ở mức 50 users. Con số chứng minh thuyết phục nhất là: khi offered load tăng 5x, throughput thực tế không những không tăng mà giảm nhẹ từ 2.66 xuống 2.39 RPS (0.90x), trong khi độ trễ P95 tăng vọt 5.24x từ 4.2s lên 22.0s. 

Theo định luật Little's Law ($L = \lambda \times W$), Effective Concurrency ở 50 users là 38.2, vượt gấp 9.5 lần số slot phục vụ song song của server (`--parallel 4`). Vì server chỉ decode tối đa 4 request cùng lúc, 34+ request còn lại phải chờ trong hàng đợi. Do đó, phần độ trễ tăng vọt ở P95 chính là Queue Time, không phải Compute Time. Nếu cần nâng cao goodput@SLO (ví dụ SLO P95 <= 5s), tôi sẽ ưu tiên tăng `--parallel` lên 8 slot và cấu hình KV cache quantization (`--cache-type-k q8_0`) để chứa thêm slot mà không tràn VRAM.
