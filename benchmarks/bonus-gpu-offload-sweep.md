# Bonus - GPU offload sweep

Host `Windows-AMD64` · backend(s) `nvidia_cuda, vulkan` ·
llama.cpp `b10488` · `threads=12` · metric `tg128`

| -ngl | tg128 (tok/s) | vs -ngl 0 | vs best |
|:--|--:|--:|--:|
| 0 | 13.6 | 1.00x | 17% |
| 8 | 16.7 | 1.23x | 21% |
| 16 | 20.6 | 1.52x | 26% |
| 24 | 38.1 | 2.81x | 48% |
| 32 | 57.2 | 4.23x | 73% |
| 99 | 78.9 | 5.83x | 100% |

Best: `-ngl 99` at 78.9 tok/s
-- 5.83x faster than CPU-only.

Where the curve flattens tells you the model ran out of layers to move. Where it
*peaks below* full offload tells you something did not fit and the accelerator
started paying to fetch weights it could not hold.

## Your finding

Full offload (`-ngl 99`) đạt tốc độ tốt nhất tuyệt đối trên máy tôi (78.9 tok/s, tăng 5.83x so với CPU thuần 13.6 tok/s). 

Vì mô hình Gemma 4 E2B UD-Q4_K_XL có kích thước 2.97 GB, hoàn toàn nằm trọn trong 4 GB VRAM của card NVIDIA GeForce RTX 3050 Ti, không xảy ra hiện tượng tràn bộ nhớ hay phải tráo đổi trọng số qua lại giữa host và device qua bus PCIe. Trong khi đó, các mức partial offload (`-ngl 8..24`) chỉ tăng tốc khiêm tốn vì vẫn phải chia sẻ tính toán trên CPU và chịu chi phí đồng bộ hóa dữ liệu trung gian qua PCIe. Toàn bộ đồ thị minh chứng rõ ràng: chuyển trọn vẹn mô hình vào bộ nhớ GPU để khai thác băng thông GDDR6 là yếu tố quyết định tốc độ decode.
