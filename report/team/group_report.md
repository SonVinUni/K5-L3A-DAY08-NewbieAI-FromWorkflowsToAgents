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

Nhóm hiểu bài toán thế nào, chia việc ra sao, thứ tự làm bốn shape.

...

## 3. Giải pháp

Mô tả ba shape nhóm viết (workflow, single, orchestrator): mỗi shape đọc gì, quyết định gì, gọi host ra sao. Chỉ rõ ranh giới code / model (tham khảo [docs/architecture.md](../../docs/architecture.md)), cách handoff được sanitize và trace ghi những event nào.

...

## 4. Kết quả

Bảng so sánh 4 shape trên bộ public (từ `src/reports/benchmark.json`): pass rate, pass^5, seed min–max, variance, cost/success, mean/p95 steps. Kèm crash safety (`src/reports/crash.json`) và, nếu có, kết quả tự kiểm trên `student_drop/`.

...

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

Lỗi gặp phải (kèm lệnh, output hoặc ảnh chụp) và cách nhóm xử lý.

...

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

...

## 8. Tài liệu tham khảo

...
