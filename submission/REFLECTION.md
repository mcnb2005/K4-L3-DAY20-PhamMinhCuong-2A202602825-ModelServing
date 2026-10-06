# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Phạm Minh Cương
**MSSV:** 2A202602825
**Cohort:** A20-K4
**Ngày submit:** 2026-10-06

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11 (AMD64)
- **CPU:** 13th Gen Intel(R) Core(TM) i7-13700H
- **Cores:** 14 physical / 20 logical
- **CPU extensions:** AVX2
- **RAM:** 31.7 GB
- **Accelerator:** Intel Iris Xe Graphics; Vulkan detected, CPU runtime used (`ngl=0`)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cpu-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi
_(Nếu dùng cloud fallback: nói rõ vì sao — RAM < 8 GB, setup fail, v.v. Không mất điểm.)_

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Máy đủ RAM cho Gemma nhưng HF CDN chặn file thứ hai nên tôi chuyển sang Qwen chính
thức. Python không có trong PATH nên dùng runtime đi kèm Codex. Gói Vulkan tải lỗi
ZIP, vì vậy tôi dùng CPU prebuilt chính thức. File Q2 tải qua mirror và được đối
chiếu SHA-256 từ Hugging Face trước khi benchmark.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3330 | 226 / 286 | 45.3 / 49.8 | 3011 / 3363 / 3363 | 22.1 |
| UD-Q2_K_XL | 0.39 | 2791 | 290 / 356 | 36.8 / 44.1 | 2606 / 3078 / 3078 | 27.2 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 0.11 GB và decode nhanh hơn 1.23×, nhưng TTFT P50 cao hơn 28%. Với cùng
câu hỏi, hai bản đều giữ cấu trúc ba ý; Q2 lệch sang monitoring/autoscaling thay vì
định nghĩa goodput trực tiếp. Tôi chọn Q2 cho serving ưu tiên throughput, còn Q4 cho
câu trả lời nhạy về chất lượng.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.53 | 16000 | 23000 | 23000 | 8.5 | 0.0% |
| 50 | 0.72 | 38000 | 55000 | 57000 | 23.5 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.34×
- **P95 tăng:** 2.39×
- **Effective concurrency ở 50 users:** 23.5 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.91 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server bão hoà trước hoặc tại 50 users: tải mời tăng 5× nhưng RPS chỉ tăng 1.34×,
trong khi P95 tăng 2.39× lên 55 s. `busy_slots=3.91/4` và 46 request deferred cho
thấy latency tăng chủ yếu là queue time. Tôi sẽ thử `--parallel 8` trước vì RAM còn
dư, rồi chỉ giữ nếu goodput tại P95 SLO tăng mà TPOT không phồng đủ để triệt tiêu lợi ích.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | local Windows, không có IaC | stub |
| N17 Data pipeline | `TOY_DOCS` tĩnh | stub |
| N18 Lakehouse | không có bảng/lakehouse ngoài | stub |
| N19 Vector + features | keyword overlap, không phải vector index | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llm: 7878.5 ms
- **stage chiếm nhiều nhất:** llm (100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM chiếm gần 100% đúng như kỳ vọng vì embed/retrieve stub chỉ mất 0.1 ms. Muốn giảm
latency 2×, tôi sẽ tối ưu decode bằng Vulkan offload và giới hạn output token, rồi đo
lại; tối ưu retrieval 0.1 ms không thể làm tổng thời gian thay đổi đáng kể.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** hạ số thread decode từ `-t 14` xuống `-t 7`

```
before:  49.1 tok/s (`-t 14`, physical-core default)
after:   50.5 tok/s (`-t 7`)
speedup: 1.03×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Điểm gãy nằm giữa 7 và 14 threads: 7 threads đạt 50.5 tok/s, còn 14 threads đã chậm
hơn khoảng 3%. Decode phải đọc lặp lại trọng số model, nên bảy worker có vẻ đã khai
thác gần hết băng thông bộ nhớ/cache hữu ích trên CPU hybrid này. Thêm worker không
tạo thêm băng thông tương ứng.

Từ 20 threads trở lên, các worker tranh cache và memory channel, đồng thời tăng chi
phí scheduling/synchronization; vì vậy 20 threads còn 36.7 tok/s và oversubscribe 40
threads chỉ còn 18.2 tok/s. Kết quả peak trước physical-core count cho thấy đây không
phải bài toán cứ thêm core là nhanh hơn, mà đã chạm giới hạn cấp dữ liệu cho decode.

---

## 6. Bonus  *(optional — tối đa 10 điểm)*

> Bỏ trống nếu không làm. Xem `docs/bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

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
- [ ] Đã push và paste public URL vào VinUni LMS **trước 23:59 (UTC+7) ngày làm lab**
- [x] **Không** commit `models/*.gguf`, `runtime/` hay `.env` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.

---

## 9. Khai báo sử dụng AI  *(xem `docs/RULES.md` §3)*

Codex được dùng để chạy và debug các script của lab, xử lý lỗi tải runtime/model,
tóm tắt các số đo thật vào report, và kiểm tra tính nhất quán trước khi nộp. Không dùng
AI để bịa số liệu; toàn bộ bảng latency/load/pipeline do script trong repo sinh ra.
