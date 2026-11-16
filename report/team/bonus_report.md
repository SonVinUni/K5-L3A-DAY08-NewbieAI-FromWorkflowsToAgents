# Bonus report: B1 · Investigator song song và Critical Path

- **Topic + nguồn (tên, năm, slide):** Google Research 2026, *Towards a Science of Scaling Agent Systems* (Slide 15 Parallelization, Slide 24 Graph scheduling, Slide 33 Latency model).
- **CLAIM của nguồn (1 câu):** Hệ thống multi-agent chỉ mang lại lợi ích về độ trễ khi các tác vụ có thể phân rã và thực thi song song độc lập; với các luồng phụ thuộc tuần tự, overhead giao tiếp làm giảm hiệu quả từ 39–70%.
- **BOUNDARY (model / dataset / setup):** MockLLM seeded cố định, bảng giá token mô phỏng (`pricing.yaml`), 14 public claims $\times$ 5 seeds, môi trường SQLite cục bộ đa luồng.
- **HYPOTHESIS trên LogiX:** Tách các tác vụ đọc độc lập (`lookup_order`, `lookup_policy`) sang thực thi song song bằng `ThreadPoolExecutor` sẽ rút ngắn đường găng (critical path) khoảng 25–40% theo định luật Amdahl mà không làm giảm pass rate hay gây xung đột dữ liệu SQLite.
- **Biến cố định / biến thay đổi:**
  - *Biến cố định:* Cùng tập claim (14 claims), cùng 5 seeds (11, 22, 33, 44, 55), cùng evaluator, cùng logic tính bồi thường (`compute_compensation`), cùng cổng host `issue_refund`.
  - *Biến thay đổi:* Topology thực thi của Investigator (Tuần tự `orchestrator` vs Song song đa luồng `orchestrator_parallel`).
- **Lệnh tái lập (copy-paste được):**
  ```bash
  python -m lab8.benchmark --shapes orchestrator,orchestrator_parallel --extra-shape orchestrator_parallel=bonus.orchestrator_parallel:run_claim --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json
  ```
- **File report:** `src/reports/bonus_benchmark.json`

---

## 1. Kết quả thực nghiệm

Số liệu trích xuất từ `src/reports/bonus_benchmark.json` (14 claims $\times$ 5 seeds = 70 trials mỗi shape):

| Shape | Pass rate | Pass^5 | Variance | Min | Max | Cost/success (VND) | Mean steps |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **`orchestrator` (Tuần tự)** | **1.000** | **1.000** | **0.000000** | 1.000 | 1.000 | 47.36 | 2.79 |
| **`orchestrator_parallel` (B1)** | **1.000** | **1.000** | **0.000000** | 1.000 | 1.000 | 38.08 | 3.64 |

* **Độ an toàn dữ liệu:** `crash_safety: 20/20`, `secret_leak_count: 0`, `approval_bypass_count: 0`.

---

## 2. Phân tích đường găng (Critical Path & Amdahl's Law)

* **Phần nào thực sự song song được trong Claims flow?**
  - Bước đọc thông tin đơn hàng (`lookup_order`) và bước tra cứu chính sách bồi thường (`lookup_policy`) là **2 tác vụ hoàn toàn độc lập** (không phụ thuộc dữ liệu của nhau), do đó song song hóa được 100%.
  - Bước tính tiền bồi thường (`compute_compensation`) bắt buộc phải tuần tự vì cần đầu vào là kết quả của `lookup_order`.
  - Bước ghi sổ (`issue_refund`) bắt buộc tuần tự với duy nhất 1 writer để bảo đảm ACID của SQLite.

* **Đo lường độ trễ theo mô hình DAG (`bonus/latency.yaml`):**
  - **Độ trễ tuần tự ($T_{seq}$):** $T_{claim} (20ms) + T_{order} (45ms) + T_{policy} (35ms) + T_{quote} (25ms) = 125ms$.
  - **Độ trễ song song ($T_{par}$):** $T_{claim} (20ms) + \max(T_{order}, T_{policy}) (45ms) + T_{quote} (25ms) = 90ms$.
  - 👉 **Hệ số tăng tốc (Speedup):** $\frac{125ms}{90ms} \approx \mathbf{1.389\times}$ (rút ngắn **38.9%** thời gian đường găng điều tra). Kết quả này hoàn toàn khớp với dự đoán của mô hình Amdahl (slide 33).

---

## 3. Kết luận

* **Ủng hộ giả thuyết:** 
  1. Chạy song song các Investigator đọc độc lập giúp tối ưu đáng kể đường găng mà vẫn duy trì **Pass rate 100%** và **Variance = 0**.
  2. Bằng chứng kiểm thử trong `tests/test_bonus_b1.py` xác nhận: Việc mỗi worker thread mở kết nối SQLite riêng giải quyết triệt để vấn đề thread safety, không gây khóa database hay trùng lặp refund.
* **Điều kiện đảo ngược:**
  - Nếu số lượng tool đọc phụ thuộc lẫn nhau tăng lên (ví dụ: cần đọc order mới biết policy nào cần tra), phần song song hóa giảm về 0, overhead tạo thread sẽ làm hệ thống chậm hơn bản tuần tự.

---

## 4. Giới hạn

- MockLLM không đo độ trễ mạng thực tế; độ trễ được tính qua mô hình DAG mô phỏng cố định từ `latency.yaml`.
- Thử nghiệm trên tập dữ liệu 14 claim public; trên quy mô hàng triệu claim đồng thời cần cơ chế Connection Pool chuyên dụng thay vì mở thread connection cục bộ.
