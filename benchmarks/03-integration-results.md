# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throug... | goodput, paged, radix | 0.0 | 0.1 | 3351.1 | 3351.2 |
| What problem does PagedAttention actually s... | paged, radix, disagg | 0.0 | 0.1 | 2961.8 | 2961.9 |
| When does splitting prefill and decode help... | disagg, radix, batching | 0.0 | 0.1 | 2875.5 | 2875.6 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **3062.8** · total **3062.9**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Goodput@SLO counts only the requests per second that met the TTFT and TPOT targets. Throughput at saturation ignores SLOs.

**What problem does PagedAttention actually solve?**

> PagedAttention stores the KV cache in non-contiguous pages, removing the internal fragmentation that wasted most GPU memory.

**When does splitting prefill and decode help?**

> Splitting prefill and decode helps because prefill is compute-bound and decode is memory-bandwidth-bound.

## Which N16-N19 pieces are real

- **N16 Cloud/IaC:** Stub (chạy local trên Windows localhost)
- **N17 Data pipeline:** Stub (danh sách documents trong bộ nhớ)
- **N18 Lakehouse:** Stub (dictionary TOY_DOCS)
- **N19 Vector + features:** Stub (keyword overlap retrieval, embed = 0.0 ms)
- **N20 Serving:** REAL (endpoint llama-server OpenAI-compatible trên cổng 8080)

Nhận xét: Giai đoạn LLM chiếm 100% tổng thời gian (3062.8 ms trên 3062.9 ms), hoàn toàn khớp với kỳ vọng vì phần retrieval chỉ là tìm kiếm từ khóa trong bộ nhớ RAM diễn ra trong 0.1 ms. Nếu cần giảm 2x độ trễ của pipeline này, tôi sẽ tấn công trực tiếp vào stage LLM bằng cơ chế Prompt Prefix Caching (giữ nguyên system prompt để tái sử dụng KV cache) và tối ưu hóa context size đưa vào prompt.
