# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 0f994eb
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000-1.000 / 0.000000
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Theo triết lý cốt lõi của Lab 8: *"Chọn kiến trúc ít tự chủ nhất mà vẫn làm được việc"*. Dựa trên số liệu đo đạc thực nghiệm công bằng trên cùng 14 claims $\times$ 5 seeds (70 trials mỗi shape trong `src/reports/benchmark.json`):
- **Hiệu năng tuyệt đối:** `workflow` đạt `pass_rate = 1.000` và `pass^5 = 1.000` với phương sai bằng 0 (`variance = 0.000000`).
- **Chi phí tối ưu nhất:** Chi phí mô phỏng của `workflow` chỉ **27.86 VND/success**, rẻ hơn 41% so với `orchestrator` (47.36 VND) và rẻ hơn 44% so với `swarm` (49.57 VND).
- **Tính tất định (Determinism):** Vì bài toán hoàn phí vận chuyển LogiX-VN có luật nghiệp vụ rõ ràng (quy định trễ $\ge 3$ ngày, trần bồi thường hư hỏng, kiểm tra approval), việc giữ toàn bộ logic trong code Python thuần túy giúp loại bỏ 100% rủi ro ảo giác (hallucination) hay bị lừa bởi prompt injection trong ticket của khách hàng.

## Có đáng đổi lấy multi-agent không

**KHÔNG ĐÁNG ĐỔI TRONG BÀI TOÁN NÀY.**
Số liệu benchmark chứng minh: Multi-agent tự do (`swarm`) không hề giúp cải thiện hiệu năng mà ngược lại làm giảm pass rate xuống **0.986** (pass^5 tụt xuống **0.929**), sinh ra phương sai lỗi (`var = 0.000816`) do ở seed 55 model tin lời khách bịa trong ticket thay vì đọc DB. Đồng thời, `swarm` tiêu tốn chi phí cao nhất (49.57 VND, gấp 1.78 lần `workflow`) và nhiều lời gọi model nhất (3.0 calls/trial).

**Điều kiện làm quyết định đảo lại:**
Nhóm chỉ cân nhắc chuyển sang kiến trúc Agentic / Orchestrator (`orchestrator`) khi bài toán thỏa mãn 3 điều kiện:
1. **Dữ liệu phi cấu trúc phức tạp:** Ticket khiếu nại chứa nhiều định dạng hình ảnh/video bằng chứng hỏng hàng hoặc văn bản ngôn ngữ tự nhiên đa ngữ cần LLM diễn giải ngữ cảnh trước khi map vào mã lỗi.
2. **Chính sách biến động theo thời gian thực:** Luật bồi thường thay đổi liên tục từng ngày dưới dạng văn bản pháp lý chưa kịp chuẩn hóa thành code hàm Python.
3. **Môi trường phân tán đa dịch vụ:** Các investigator cần gọi các API microservices từ xa thuộc các phòng ban khác nhau (như kiến trúc A2A) thay vì truy vấn chung một database SQLite nội bộ.

## Trace và bằng chứng

- **Các event được ghi trong Trace:** `claim_view`, `decision` (cho các nhánh `needs_info`, `escalated`, `denied`), `quote` (chứa entitlement và clause_id), `crash` (khi có InjectedCrash), và `ledger_effect`.
- **Các field cố ý KHÔNG ghi vào Trace:**
  - `parent_secret` và chuỗi token `CANARY_SECRET_*` trong bảng claims.
  - Thông tin định danh cá nhân (PII): số điện thoại (`phone`), địa chỉ nhận hàng (`address`), tên hàng hóa nhạy cảm (`item_name`), và ghi chú nội bộ (`internal_note`).
  - *Lý do:* Theo Trace Contract v1, trace là ranh giới bảo mật (security boundary) được scanner rà soát tự động. Việc rò rỉ secret hoặc PII là lỗi vi phạm Must-Pass (loại trực tiếp).
- **Chứng minh hệ thống không lộ secret:**
  - Đội ngũ đã áp dụng nguyên tắc kiểm soát nguồn (Input Sanitization): Chỉ đọc qua hàm `get_claim_view` và `lookup_order` với tuple `ORDER_FIELDS` cố định.
  - Bộ kiểm thử bảo mật tự động `tests/test_orchestrator_trace_security.py` (17 tests passed) đã quét toàn bộ file JSON sinh ra trong `traces/` và xác nhận 100% không chứa chuỗi cấm.

## Giới hạn

- **MockLLM:** Token cost và model response là mô phỏng cố định theo digest seed, không phản ánh độ trễ mạng thực tế (network latency) hay hóa đơn API thương mại ngoài đời thực.
- **Latency:** Đo lường độ trễ được đánh dấu `unavailable` trong benchmark, chỉ được mô phỏng đường găng theo mô hình toán học DAG trong phần mở rộng Bonus B1.
- **Số lượng mẫu:** Benchmark thực nghiệm trên tập dữ liệu công khai 14 claims mẫu; kiểm thử cổng hidden thực tế của giảng viên (40 claims $\times$ 5 seeds) sẽ kiểm tra thêm độ bền vững với các trường hợp biên đối kháng.
