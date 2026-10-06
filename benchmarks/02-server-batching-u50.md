# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 14 samples over
60s at 5s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.96 of 4 slots (99%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 16888 |

Highest sampled value was **3.96 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Your observation

Peak batch width đạt 3.96 trên 4 slots (99% công suất slot decode), chứng minh continuous batching đã hoạt động hết công suất thiết kế: scheduler của llama-server liên tục gộp đủ 4 request vào mỗi bước decode chung.

Con số này hoàn toàn khớp và giải thích số liệu Effective Concurrency (38.2) trong `02-server-results.md`. Hai chỉ số không mâu thuẫn mà đo hai khía cạnh khác nhau: gauge `n_busy_slots_per_decode` đo số slot đang trực tiếp tính toán trong vòng lặp decode (bị chặn trên bởi `--parallel 4`), còn Effective Concurrency đo tổng số request đang nằm trong toàn bộ hệ thống (gồm cả 4 request đang chạy và đỉnh điểm 46 request nằm chờ trong hàng đợi `requests_deferred`). Điều này chứng minh tuyệt đối hiện tượng thắt cổ chai do hàng đợi khi hệ thống quá tải.
