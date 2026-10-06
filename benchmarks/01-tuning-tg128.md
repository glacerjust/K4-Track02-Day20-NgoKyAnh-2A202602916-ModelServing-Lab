# 01 - Tune: thread-count sweep

Model `gemma-4-E2B-it-UD-Q4_K_XL.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **12 physical · 16 logical** cores · `ngl=99` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 80.8 | 97% |
| 6 | 83.0 | 100% |
| 12 | 83.1 | 100% |
| 16 | 83.2 | 100% |
| 32 | 83.1 | 100% |

**Best**: `-t 16` at 83.2 tok/s
**Slowest tested**: `-t 1` at 80.8 tok/s (1.03x spread)
**Against the physical-core default** (`-t 12`, 83.1 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=16 make bench
```

## Your explanation

Đồ thị throughput theo số thread ở đây gần như đi ngang phẳng (từ 80.8 tok/s ở 1 thread lên 83.2 tok/s ở 16 threads, chênh lệch chỉ ~3%). 

Nguyên nhân theo cơ chế phần cứng: Máy tính có GPU NVIDIA RTX 3050 Ti và cờ `-ngl 99` đã đẩy toàn bộ các lớp của mô hình Gemma 4 E2B vào GPU VRAM. Khi đó, toàn bộ phép nhân ma trận (GEMV/GEMM) trong quá trình decode đều do GPU CUDA cores và GPU memory bandwidth xử lý. Các luồng CPU (`-t`) chỉ đóng vai trò điều phối, chuẩn bị token và gọi kernel GPU, do đó số lượng CPU thread hầu như không tạo ra nút thắt cổ chai và không có sự sụt giảm mạnh do oversubscription CPU. Điểm tối ưu nhẹ đạt được ở 16 threads (bằng số logical cores), tận dụng tối đa khả năng scheduling của hệ điều hành Windows.
