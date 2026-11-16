# Báo cáo cá nhân — 2A202603629_TranNguyenAnhPhong

## 1. Thông tin thành viên

- **Họ tên:** Trần Nguyễn Anh Phong
- **MSSV:** 2A202603629
- **Email:** anhphong7cpct@gmail.com
- **GitHub:** https://github.com/anhphongtrng
- **Nhóm:** NewbieAI — Lab 08: From Workflows to Agents

## 2. Phần việc phụ trách

Phụ trách **Shape B — Single Agent**, tập trung vào `src/lab8/shapes/single.py` và kiểm thử riêng trong `src/tests/test_single_agent_custom.py`.

- Triển khai hàm `run_claim()` theo contract dùng chung và hàm `decide()` cho vòng ReAct giới hạn tối đa 8 bước.
- Cho agent chọn các **read tools** để đọc claim, xác minh đơn hàng, tra policy, lịch sử refund và tính compensation; không cho model gọi write tool.
- Để host thực hiện `durability.issue_refund()` với **quote đầy đủ**, đảm bảo approval, idempotency và kiểm soát hoàn tiền theo đơn hàng.
- Ghi nhận `Budget` (model calls, tool calls, tokens, chi phí VND mô phỏng), tạo trace JSON và giữ an toàn dữ liệu nhạy cảm.
- Xây dựng test bổ sung cho Shape B và phối hợp đánh giá kết quả benchmark của nhóm.

## 3. Đóng góp cụ thể

### 3.1. Triển khai Single Agent

- Thay thế stub ban đầu bằng luồng mở kết nối SQLite qua `lab8.db.connect()`, thực thi `run_react()` và tạo `RunResult` bằng `lab8.shapes.common.finish()`.
- Xây dựng `decide(observation)` theo các phản hồi tool: `get_claim_view` → `lookup_order` → `lookup_policy` → `lookup_refund_history` → `compute_compensation`; kết thúc sớm khi claim không tồn tại, thiếu mã đơn hoặc không được phép xem đơn.
- Lưu kết quả đọc cần thiết trong state cục bộ do runtime chỉ giữ phản hồi tool cuối cùng; không đưa dữ liệu nhạy cảm vào trace.
- Sử dụng dữ liệu order và policy hiện hành để tính compensation, không suy luận số tiền từ ticket do khách hàng nhập.
- Thực hiện refund ở tầng host thay vì từ `decide()`, truyền `crash_at` và không nuốt `InjectedCrash`; xử lý thêm các trường hợp conflict/lỗi runtime trong bản code đã review.
- Tích hợp `lab8.mock_llm.complete()` để tạo thông báo ngắn; hàm này tự cập nhật số model calls, token và chi phí mô phỏng trong `Budget`. Model không quyết định chính sách hay số tiền hoàn.

### 3.2. Test đã xây dựng và thực hiện

- **Test tự viết:** `src/tests/test_single_agent_custom.py` — phiên bản mở rộng có kiểm thử nhiều public claims theo các seed, kiểm tra kết quả xử lý, Budget/trace và retry idempotency. **Chưa có log xác nhận lượt chạy riêng của file này trong trao đổi hiện tại; cần bổ sung sau khi chạy.**
- **Host tests:** `python -m pytest -q` — **54 passed, 248 deselected**.
- **Student tests riêng Shape B:** `python -m pytest -q -m student -o addopts= -k "single and not workflow and not orchestrator and not swarm"` — **62 passed, 240 deselected**.
- **Crash safety suite:** `python -m lab8.crash_test --count 20 --out reports/crash.json` — **crash_safety 20/20**. Đây là kết quả suite chung; không khẳng định tất cả 20 case đều là của Shape B.

### 3.3. Quyết định kỹ thuật

1. Giữ cùng business rules/contract với các shape khác để benchmark có thể so sánh.
2. Dùng `run_react(max_steps=8)` và chỉ cung cấp read tools cho `decide()`; quote và refund là ranh giới giữa agent và host.
3. Không tự trừ refund trước đó khỏi quote: host chịu trách nhiệm thực thi per-order cap và idempotency.
4. Sử dụng MockLLM có seed để mô phỏng usage, không dùng API model thật hoặc API key.
5. Dùng `Trace` và `finish()` có sẵn để giảm sai lệch contract; không ghi ticket thô, bí mật hoặc thông tin riêng tư vào trace.

### 3.4. Commit chính

- **Commit triển khai Shape B:** feat: implement Shape B ReAct claim processing
- **Commit hardening sau review:** fix(single-agent): harden refund conflict and runtime error handling
- **Commit thêm test riêng:** test(single-agent): add custom tests for claim processing and idempotency

## 4. Bằng chứng

| Bằng chứng | Kết quả / vị trí |
|---|---|
| Xử lý claim mẫu | `C-001`, order `O-107`: `status=resolved`, `proposal_amount=35000` VND |
| Usage lần chạy mẫu | `model_calls=1`, `tool_calls=5`, `input_tokens=25`, `output_tokens=11`, `cost_estimate=51.38` |
| Run ID | `R-single-C-001-s11` |
| Trace | `traces/R-single-C-001-s11.json` (file local) |
| Trace kiểm tra nhanh | 4 events, danh sách events hợp lệ; không tìm thấy các khóa `CANARY_SECRET_`, `parent_secret`, `raw_headers`, `internal_note`, `phone`, `address`, `item_name` trong trace mẫu |
| Host tests | 54/54 passed trong lượt chạy đã báo |
| Student tests Shape B | 62/62 passed trong lượt chạy đã báo |
| Crash suite | 20/20 passed trong lượt chạy đã báo |
| Test tự viết | `src/tests/test_single_agent_custom.py` — pass |

**Kết quả benchmark nhóm được cung cấp:** Shape B (`single`) đạt **pass rate 1.000**, **Pass^5 1.000**, **seed min–max 1.000–1.000**, **variance 0.000000**, **cost/success 46.37 VND mô phỏng**, **mean/p95 steps 3.64/4**. Đối chiếu: Workflow đạt cost/success 27.86; Orchestrator 47.36; Swarm 49.57. Đây là kết quả benchmark của nhóm, không phải số liệu tự chạy riêng trong báo cáo cá nhân.

## 5. Công cụ AI đã dùng

- **Công cụ:** ChatGPT.
- **Mục đích:** Giải thích `RunResult`, `Budget`, `Trace`, SQLite, ReAct và read tools; hỗ trợ xây dựng/chỉnh sửa `single.py`; gợi ý test, phân tích lỗi và diễn giải số liệu benchmark.
- **Tự kiểm tra:** Đọc và đối chiếu code với API trong các file `runtime.py`, `tools.py`, `schemas.py`, `db.py`, `durability.py`, `mock_llm.py`; chạy `py_compile`, host tests, student tests, crash suite; chạy thử claim `C-001`, kiểm tra usage và nội dung trace.
- **Giới hạn sử dụng AI:** Không sử dụng API model thật trong lab; lời gọi model của Shape B đi qua `MockLLM`. Không dùng kết quả AI thay thế quyết định refund/approval. Không coi nội dung AI sinh ra là bằng chứng nếu chưa đối chiếu với test/log thực tế.

Việc khai báo và tự xác minh này tuân theo `RULES.md`: cho phép dùng AI để giải thích, gợi ý code/test và đọc trace nhưng không được bịa số liệu, commit hoặc bằng chứng.

## 6. Tự đánh giá

Tôi đã hoàn thành phần triển khai chính của Shape B và xác minh bằng các bộ kiểm thử công khai: **54 host tests**, **62 student tests riêng Shape B** và **20/20 crash safety**. Luồng xử lý claim `C-001` có kết quả đúng, ghi nhận được số tool/model calls, token và chi phí mô phỏng. Tôi cũng đã xây dựng file test riêng nhằm mở rộng phạm vi kiểm tra từ một claim sang nhiều claim và tình huống retry.

**Điểm mạnh:** Tuân thủ hợp đồng đầu vào/đầu ra, không trao quyền ghi refund cho model, phân biệt dữ liệu khách hàng khai với dữ liệu order đáng tin cậy, ghi được metric và trace, có kiểm thử lặp lại.

**Hạn chế:** Chiến lược `decide()` hiện là quy tắc Python theo observation, nên tính tự chủ của agent còn hạn chế; `MockLLM` chủ yếu sinh thông báo sau xử lý. Chi phí Shape B cao hơn Workflow trong benchmark nhóm dù cùng đạt pass rate 100%. Kiểm tra từ khóa trace chỉ là kiểm tra nhanh, không thay thế toàn bộ audit bảo mật.