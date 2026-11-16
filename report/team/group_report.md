# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.


## 1. Thông tin nhóm

- Tên nhóm:
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`):
##  Phân bổ công việc nhóm 4 người

| Thành viên | Nhiệm vụ chính | Trách nhiệm cụ thể | Tiêu chí phụ trách |
| :--- | :--- | :--- | :--- |
| **Người 1** | **Shape A (Workflow) + Báo cáo chung** | - Viết `src/lab8/shapes/workflow.py`<br>- Chạy smoke, giữ cổng host `pytest -q`<br>- Chạy benchmark, crash suite khi các shape xong<br>- Viết `group_report.md`, `decision_adr.md`, `TEAM.md` | C1 (Workflow), C6, C7 |
| **Người 2** | **Shape B (Single Agent)** | - Viết `src/lab8/shapes/single.py`<br>- Thiết kế hàm `decide()` cho vòng lặp `run_react` (chỉ chọn tool đọc, không gọi write tool)<br>- Quản lý Budget (đếm model calls, token, steps ≤ 8)<br>- Viết test cho Shape B trong `src/tests/` | C1 (Single Agent), C8 |
| **Người 3** | **Shape C (Core Flow & Durability)** | - Dựng khung điều phối `run_claim` trong `orchestrator.py`<br>- Tích hợp `run_readonly_investigators`<br>- Tính tiền từ kết quả order view và gọi `durability.issue_refund` với `crash_at`<br>- Xử lý các nhánh status (`needs_info`, `escalated`, `denied`, `resolved`) | C1 (Orchestrator), C3, C4 |
| **Người 4** | **Shape C (Handoff, Security/Trace & Test)** | - Thiết kế Handoff chuẩn allowlist (`make_handoff`, lọc facts không dính `parent_secret`/PII)<br>- Đảm bảo Trace Contract v1 (không leak secret/phone/address/internal note)<br>- Viết bộ test riêng cho Shape C vào `src/tests/` (bắt buộc để có điểm C8)<br>- Test kiểm tra rò rỉ secret và crash idempotency của Shape C | C5 (Handoff & Trace an toàn), C8 |


## 2. Mục tiêu và cách tiếp cận

Bài toán là xử lý claim hoàn phí vận chuyển trên cùng một bộ dữ liệu nhưng triển khai bằng bốn kiến trúc: workflow, single agent, orchestrator và swarm. Nhóm giữ phần quyết định tiền, approval, giới hạn hoàn theo order, idempotency và ghi ledger ở code/host; model chỉ hỗ trợ các thao tác đọc hoặc soạn câu trả lời. Nhóm chia việc theo shape và phần an toàn dữ liệu, sau đó chạy cùng corpus 14 claim và 5 seed để so sánh công bằng.

## 3. Giải pháp

Shape A (workflow) dùng các bước Python cố định: đọc claim bằng `get_claim_view`, kiểm tra `order_id`, gọi `lookup_order`, tính quote bằng `compute_compensation` và nếu quote dương thì gọi `durability.issue_refund` với quote đầy đủ. Shape này không dùng model để chọn số tiền nên không bị ảnh hưởng bởi nội dung ticket hay seed.

Shape B (single) dùng `run_react` với giới hạn 8 bước. Agent chỉ được chọn các read tool; sau khi có quote, shape chuyển sang host để kiểm approval, cap theo order, idempotency và ghi refund. Shape C (orchestrator) tạo handoff bằng allowlist, chạy investigator chỉ đọc qua `run_readonly_investigators`, rồi orchestrator dùng order view để tính quote và gọi host. Handoff không chứa parent secret, raw headers hay dữ liệu ngoài allowlist. Trace ghi những event cần thiết như claim view đã lọc, investigator/handoff, quote và decision; các field nhạy cảm không được ghi.

## 4. Kết quả

Số liệu dưới đây lấy từ `src/reports/benchmark.json`, chạy trên 14 claim × 5 seed:

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean / p95 steps |
| :--- | ---: | ---: | :--- | ---: | ---: | ---: |
| workflow | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 27.86 | 3.79 / 4 |
| single | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 46.37 | 3.64 / 4 |
| orchestrator | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 47.36 | 2.79 / 3 |
| swarm | 0.986 | 0.929 | 0.929–1.000 | 0.000816 | 49.57 | 2.80 / 3 |

Workflow, single và orchestrator đều đạt toàn bộ trial public. Swarm có một trial sai ở seed 55 do điểm yếu cố ý tin số ngày trong ticket. `src/reports/crash.json` cho `crash_safety = 20/20`. Quyết định chọn workflow được giải thích trong `decision_adr.md`; ablation song song investigator được ghi trong `research_card.md` và bonus report.

## 5. Khó khăn và cách xử lý

Khó khăn đầu tiên là phân biệt môi trường Windows với môi trường POSIX: Git Bash dùng `.venv/Scripts/activate`, không dùng `.venv/bin/activate`. Sau khi tạo lại virtual environment đúng cách, smoke test và cổng host đều chạy được. Khi triển khai workflow, các lỗi dễ gặp là tự trừ refund cũ, ghi raw claim/order vào trace hoặc bắt crash làm T12 không còn thấy crash. Nhóm đối chiếu business rules/contracts, chạy test T11/T12, chạy leak/security tests và chạy lại benchmark thay vì sửa số liệu thủ công.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` và các báo cáo cá nhân trong `report/team/`. Phần workflow, integration ban đầu, benchmark/crash và báo cáo chung do Võ Sỹ Thái Sơn phụ trách. Shape single và test riêng do thành viên phụ trách Shape B thực hiện. Shape orchestrator, handoff/security và bonus B1 do các thành viên phụ trách tương ứng thực hiện.

## 7. Hạn chế và hướng cải thiện

MockLLM là mô phỏng seeded, không đại diện cho model sản xuất; cost là giá mô phỏng chứ không phải hóa đơn API. Latency được ghi là `unavailable`, còn phần bonus dùng critical-path model mô phỏng. Bộ public chỉ có 14 claim nên chưa đủ để khẳng định kết quả trên mọi loại ticket. Nếu task thực tế cần xử lý ngôn ngữ mơ hồ hoặc có nhiều bước độc lập, multi-agent có thể đáng cân nhắc; còn với flow nghiệp vụ cố định, workflow có lợi thế về chi phí, khả năng kiểm tra và variance.

## 8. Tài liệu tham khảo

- `README.md`, `CHECKPOINTS.md`, `RUBRIC.md` của Lab 8.
- `docs/business_rules.md`, `docs/contracts.md`, `docs/architecture.md`, `docs/metrics.md`.
- `src/reports/benchmark.json`, `src/reports/crash.json` và `src/reports/bonus_benchmark.json`.
- Google Research 2026, *Towards a Science of Scaling Agent Systems*, dùng cho ablation B1 trong `report/team/bonus_report.md`.
