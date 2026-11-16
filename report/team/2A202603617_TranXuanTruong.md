# Báo cáo cá nhân —  2A202603617_TranXuanTruong

## 1. Thông tin thành viên

- Họ tên: Trần Xuân Trường
- MSSV: 2A202603617
- Email: truongxuan2834@gmail.com
- GitHub: [XuanTruong28]([(https://github.com/XuanTruong28]).

## 2. Phần việc phụ trách

- **Shape C (Orchestrator) — Core Flow & Durability**:
  - Chịu trách nhiệm chính về kiến trúc và luồng điều phối của Shape C trong `src/lab8/shapes/orchestrator.py`.
  - Phối hợp với Người 4 (Handoff & Security) để hoàn thiện toàn bộ quy trình từ tiếp nhận claim, điều phối investigator chỉ đọc, tính toán bồi thường và gọi cổng host an toàn.
- **Nghiên cứu mở rộng (Bonus Topic B1 — Investigator song song & Critical Path)**:
  - Chủ trì triển khai nghiên cứu mở rộng B1 theo lý thuyết của Google Research 2026 (*Towards a Science of Scaling Agent Systems*).
  - Xây dựng biến thể `bonus/orchestrator_parallel.py`, mô hình độ trễ `bonus/latency.yaml`, test suite `tests/test_bonus_b1.py` và hoàn thành báo cáo `report/team/bonus_report.md`.

## 3. Đóng góp cụ thể

- **Commit chính (Git hash):** `0f994eb` (*hàm orchestrator và test của thành viên 3 (mục C ) trong team*) cùng các commit triển khai Bonus B1.
- **Hiện thực hàm `run_claim` trong `src/lab8/shapes/orchestrator.py`:**
  - Khởi tạo session và trace, mở và đóng kết nối SQLite an toàn với khối `try ... finally: conn.close()`.
  - Phân nhánh trạng thái nghiệp vụ chuẩn xác theo hợp đồng `RunResult`:
    - Claim không tồn tại $\rightarrow$ `status="failed"`, `proposal_amount=0`.
    - Claim thiếu `order_id` $\rightarrow$ `status="needs_info"`, `proposal_amount=0`.
    - Đơn hàng không thuộc quyền sở hữu của principal $\rightarrow$ `status="escalated"`, `proposal_amount=0`.
  - Tích hợp điều tra viên qua `run_readonly_investigators` và trích xuất quote bồi thường hợp lệ từ `compute_compensation`.
  - Nhánh Quote $\le 0 \rightarrow$ trả về `status="denied"`, tuyệt đối không gọi `issue_refund`.
  - Nhánh Quote $> 0 \rightarrow$ gọi `durability.issue_refund` với quote đầy đủ và truyền tham số `crash_at`.
  - Xử lý lỗi crash: Bắt `InjectedCrash`, ghi flush trace và re-raise lại ngoại lệ theo đúng yêu cầu kiểm thử của hệ thống.
  - Chuyển tiếp kết quả từ host và trả về qua hàm chuẩn `finish()`.
- **Triển khai kiến trúc song song đa luồng (Bonus B1):**
  - Xây dựng `src/bonus/orchestrator_parallel.py` sử dụng `ThreadPoolExecutor` để thực thi song song các tác vụ đọc độc lập (`lookup_order`, `lookup_policy`).
  - Thiết kế cơ chế Thread-safe: mỗi worker thread mở kết nối SQLite riêng, tránh hoàn toàn lỗi database lock trên Windows.
  - Xây dựng mô hình đường găng DAG (Critical Path) chứng minh tốc độ xử lý nhanh hơn 38.9% theo định luật Amdahl.
- **Viết test kỹ thuật độc lập:**
  - Tạo file `src/tests/test_n3_orchestrator.py` với test case `test_orchestrator_needs_info_on_missing_order` kiểm thử trường hợp claim thiếu mã đơn hàng (`C-004`).
  - Tạo file `src/tests/test_bonus_b1.py` với 2 test case chứng minh tính nhất quán 100% giữa bản song song và tuần tự, cùng tính lũy nghiệm (idempotency) khi chạy lại.

## 4. Bằng chứng

- **Commit hash:** `0f994eb` và các commit bonus.
- **Kết quả kiểm thử tự động:**
  - Ma trận test học viên của Orchestrator: Đạt **62 passed / 62 tests (100%)** (`pytest -k orchestrator`).
  - Test cá nhân độc lập: Đạt **1 passed** (`pytest tests/test_n3_orchestrator.py`).
  - Test Bonus B1: Đạt **2 passed** (`pytest tests/test_bonus_b1.py`).
  - Test Bảo mật & Crash (Người 4): Đạt **23 passed** (`test_orchestrator_crash_resume.py` và `test_orchestrator_trace_security.py`).
  - Cổng Host chung: Đạt **54 passed**.
- **Kiểm thử tính lũy nghiệm (Idempotency & Durability):**
  - Chạy lặp lại claim `C-001` bằng lệnh `python -m lab8.run --shape orchestrator --claim-id C-001 --seed 11 --db data/test_n3.sqlite`. Kết quả duy nhất 1 dòng ledger refund (35.000 VND) và `committed_audit_count: 1`, không bị nhân đôi tiền khi chạy lại.
- **Bằng chứng Benchmark Bonus B1:**
  - File `src/reports/bonus_benchmark.json` ghi nhận: Pass rate 1.000, Pass^5 1.000, Variance 0.000, Speedup đường găng 1.389x.

## 5. Công cụ AI đã dùng

- **Công cụ:** Trợ lý AI (Google DeepMind / Antigravity).
- **Mục đích sử dụng:**
  - Tra cứu và phân tích các hợp đồng contract trong `docs/contracts.md` và luật nghiệp vụ trong `docs/business_rules.md`.
  - Bóc tách kiến trúc từ file mẫu `swarm_reference.py`, loại bỏ điểm yếu `REFERENCE_WEAKNESS` để xây dựng luồng Orchestrator chuẩn xác.
  - Hỗ trợ thiết kế giải pháp đa luồng với `ThreadPoolExecutor` và mô hình Critical Path DAG cho Bonus B1.
  - Xây dựng cấu trúc các test case độc lập bằng Pytest.
- **Cách tự kiểm tra kết quả:**
  - Tự chạy toàn bộ test suite trên terminal máy cá nhân (Windows PowerShell, Python 3.14).
  - Tự verify trực tiếp file trace JSON và database SQLite để đảm bảo không rò rỉ secret và dữ liệu được ghi đúng ledger.
  - Chạy benchmark thực nghiệm 140 trials so sánh hai biến thể song song và tuần tự.

## 6. Tự đánh giá

- **Mức độ hoàn thành:** 100% yêu cầu được giao cho Shape C Core, tiêu chí đóng góp C8, và hoàn thành xuất sắc Topic Bonus B1 (+10 điểm).
- **Tuân thủ quy chế:** Đảm bảo toàn bộ các điều kiện Must-pass (không bypass approval, không lộ secret, an toàn khi crash).
- **Đánh giá:** Hoàn thành xuất sắc nhiệm vụ.
