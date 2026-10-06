# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Ngô Kỳ Anh
**MSSV:** 2A202602916
**Cohort:** A20-K4
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11
- **CPU:** 12th Gen Intel(R) Core(TM) i5-12500H
- **Cores:** 12 physical / 16 logical
- **CPU extensions:** AVX2
- **RAM:** 15.7 GB
- **Accelerator:** NVIDIA GeForce RTX 3050 Ti Laptop GPU, 4096 MiB
- **llama.cpp asset đã tải:** llama-b10488-bin-win-cuda-cu12.4-x64.zip
- **Model đã dùng:** Gemma 4 E2B (`LAB_MODEL=gemma4-e2b`)
- **Quantization:** UD-Q4_K_XL + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi
*(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)*

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Máy tính có sẵn 16 GB RAM và card rời RTX 3050 Ti nên setup diễn ra thuận lợi. Trên Windows PowerShell, script lab.ps1 gặp lỗi cú pháp do ký tự em-dash UTF-8 trong file script; sau khi chuẩn hóa về mã ASCII và cấu hình Python đọc UTF-8, toàn bộ stack setup, benchmark và load test hoạt động ổn định và tự động nhận diện CUDA backend.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| UD-Q4_K_XL | 2.97 | 5518 | 600 / 831 | 13.5 / 14.5 | 1209 / 1691 / 1691 | 74.3 |
| UD-Q2_K_XL | 2.24 | 5886 | 645 / 1210 | 13.0 / 14.1 | 1459 / 2037 / 2037 | 77.0 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Bản 2-bit decode nhanh hơn 3.6% (77.0 vs 74.3 tok/s) và tiết kiệm 0.73 GB, nhưng không đáng đánh đổi. Khi test cùng prompt, 2-bit trả lời kém mạch lạc và dễ cụt ý do mất mát trọng số quá lớn, trong khi 4-bit vừa vặn VRAM và chất lượng vượt trội.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|
| 10 | 2.66 | 2700 | 4200 | 5200 | 7.6 | 0.0% |
| 50 | 2.39 | 17000 | 22000 | 24000 | 38.2 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 0.90x
- **P95 tăng:** 5.24x
- **Effective concurrency ở 50 users:** 38.2 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.96 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hòa hoàn toàn ở 50 users khi RPS giảm nhẹ (0.90x) nhưng P95 tăng vọt 5.24x. Theo Little's Law, concurrency đạt 38.2 vượt xa 4 decode slots, chứng minh độ trễ tăng thêm là Queue Time. Để nâng goodput@SLO, tôi sẽ tăng `--parallel` lên 8 slots trước tiên nhằm mở rộng dung lượng xử lý đồng thời.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | Localhost Windows stack | stub |
| N17 Data pipeline | In-memory document list | stub |
| N18 Lakehouse | TOY_DOCS dictionary | stub |
| N19 Vector + features | Keyword overlap search | stub |
| N20 Serving | llama-server | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llm: 3062.8 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck tuyệt đối (100%), khớp với kỳ vọng vì retrieval là in-memory (0.1 ms). Để giảm latency 2x, tôi sẽ áp dụng Prompt Prefix Caching trên llama-server để tái sử dụng KV cache của context tài liệu.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** Chuyển đổi quantization từ UD-Q4_K_XL sang UD-Q2_K_XL (giảm kích thước mô hình từ 2.97 GB xuống 2.24 GB)

```
before:  74.3 tok/s
after:   77.0 tok/s
speedup: 1.04x
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Trong pha autoregressive decode của LLM, mỗi token sinh ra đòi hỏi bộ xử lý phải nạp tuần tự toàn bộ trọng số của mô hình từ bộ nhớ vào cache. Vì phép tính ma trận trên từng token có cường độ số học thấp (arithmetic intensity thấp), tốc độ decode bị giới hạn trực tiếp bởi băng thông bộ nhớ (Memory Bandwidth bound) chứ không phải năng lực tính toán FLOPs.

Khi chuyển từ UD-Q4_K_XL (2.97 GB) sang UD-Q2_K_XL (2.24 GB), dung lượng mô hình giảm bớt 0.73 GB (tương đương giảm ~24.6% lượng byte cần truyền qua bus bộ nhớ VRAM cho mỗi token). Nhờ giảm áp lực bus bộ nhớ, tốc độ decode tăng từ 74.3 lên 77.0 tok/s (đạt speedup 1.04x). Mặc dù tốc độ tăng lên, chất lượng ngữ nghĩa của 2-bit bị suy giảm rõ rệt, do đó đối với GPU 4 GB VRAM thì 4-bit vẫn là điểm cân bằng tối ưu nhất.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** B2 (sweep-gpu) & B5 (C6 - GPU Offload Acceleration)

**Numbers:**

```
before:  13.6 tok/s (-ngl 0, pure CPU)
after:   78.9 tok/s (-ngl 99, RTX 3050 Ti GPU)
speedup: 5.83x
```

**Điều này nói lên gì mà deck chưa nói:**

Tốc độ tăng trưởng không hề tuyến tính theo số layer offload. Ở giai đoạn đầu (`-ngl 0` đến `-ngl 16`), speedup chỉ tăng từ 13.6 lên 20.6 tok/s (tăng 1.5x) vì phần lớn mô hình vẫn nằm trên CPU, bus PCIe liên tục phải trung chuyển activation giữa host RAM và GPU VRAM (overhead đồng bộ hóa CPU-GPU). Tuy nhiên, từ `-ngl 24` trở lên và đạt đỉnh ở `-ngl 99` (toàn bộ model nằm trọn trong 4 GB VRAM của RTX 3050 Ti), tốc độ tăng vọt lên 78.9 tok/s (gấp 5.83x).

Điều này chứng minh: Partial offload chỉ đem lại lợi ích hạn chế do bị nghẽn băng thông truyền tải PCIe giữa CPU và GPU. Chỉ khi toàn bộ hoặc gần như toàn bộ weights nằm trong VRAM để tận dụng trọn vẹn băng thông GDDR6 nội bộ của GPU, hiệu năng decode mới bứt phá tối đa.

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

Điều làm tôi ngạc nhiên nhất là hiện tượng thắt cổ chai do hàng đợi (Queue Time) diễn ra quá rõ rệt: ở 50 users, độ trễ P95 tăng hơn 5 lần trong khi RPS lại giảm nhẹ, minh chứng rất trực quan cho định luật Little's Law và sự khác biệt sống còn giữa Raw Throughput và Goodput@SLO.

---

## 8. Self-check trước khi push

- [x] `hardware.json` committed
- [x] `models/active.json` committed
- [x] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [x] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [x] `benchmarks/02-server-results.md` committed (`make load-report`)
- [x] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [x] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [x] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [x] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [x] 5 screenshots trong `submission/screenshots/`
- [x] `make verify` → **exit 0**
- [x] Repo tên đúng mẫu `K4-L3-DAY20-HoVaTen-MSSV-ModelServing` (xem `docs/SUBMISSION.md`)
- [x] Repo GitHub ở chế độ **public**
- [x] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

Sử dụng AI Assistant (Antigravity) để hỗ trợ phân tích định dạng báo cáo, giải thích cơ chế Memory Bandwidth / Little's Law và hướng dẫn sửa lỗi encoding UTF-8 trên Windows.
