# 01 - Measure: latency baseline

Model `Gemma 4 E2B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=12` `ngl=99` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `UD-Q4_K_XL` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 5518 | 600 / 831 | 13.5 / 14.5 | 1209 / 1691 / 1691 | 74.3 |
| UD-Q2_K_XL | 2.24 | 5886 | 645 / 1210 | 13.0 / 14.1 | 1459 / 2037 / 2037 | 77.0 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.04x faster** than `UD-Q4_K_XL` here, for 0.73 GB less on disk.

## Your observation

Bản 2-bit (UD-Q2_K_XL) tiết kiệm được 0.73 GB bộ nhớ (2.24 GB so với 2.97 GB) và tốc độ decode nhanh hơn khoảng 3.6% (77.0 tok/s so với 74.3 tok/s, TPOT P50 giảm từ 13.5 ms xuống 13.0 ms). Mức cải thiện này phản ánh đúng đặc tính autoregressive decode bị thắt nút cổ chai bởi memory bandwidth: mô hình nhẹ hơn làm giảm số byte cần đọc qua bus bộ nhớ cho mỗi token sinh ra.

Tuy nhiên, bản 2-bit hoàn toàn không đáng để đánh đổi trong thực tế phục vụ người dùng. Khi nén sâu xuống 2-bit, độ suy giảm chất lượng biểu diễn trọng số rất nặng nề, dẫn đến câu trả lời dễ bị cụt, lặp ý hoặc sai lệch logic. Bản 4-bit (UD-Q4_K_XL) hoàn toàn nằm vừa trong 4 GB VRAM của GPU RTX 3050 Ti và đạt tốc độ 74.3 tok/s (rất mượt mà), đồng thời bảo toàn độ mạch lạc và chính xác ngữ nghĩa vượt trội.
