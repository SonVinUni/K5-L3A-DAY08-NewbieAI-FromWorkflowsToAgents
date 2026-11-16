# Báo cáo cá nhân — Trần Quốc Đông

## 1. Thông tin thành viên

- Họ tên: Trần Quốc Đông.
- MSSV: 2A202603643.
- Lớp: L3A.
- Email: dongtranquoc951@gmail.com.
- GitHub: [dongtranquoc951](https://github.com/dongtranquoc951).
- Nhánh làm việc: `dongtq`.
- Ngày kiểm thử ghi nhận: 16/11/2026.

## 2. Phần việc phụ trách

Theo phân công nhóm bốn người, tôi phụ trách vai trò: handoff, bảo mật trace và kiểm thử Shape C — Orchestrator. Phần đóng góp đã thực hiện gồm kiểm thử Trace Contract v1, phát hiện rò rỉ secret/PII và kiểm thử crash rồi resume không hoàn tiền hai lần.

Tôi phối hợp với người phụ trách luồng `run_claim`, investigator, tính tiền và gọi host ghi refund. Hai bộ test của tôi được viết trước khi Shape C hoàn thiện, sau đó chạy lại với code đã tích hợp. Đóng góp kỹ thuật cá nhân của tôi được ghi nhận trong hai file test.

Phần việc này cung cấp bằng chứng cho C5 (handoff và trace an toàn), C8 (đóng góp cá nhân), đồng thời hỗ trợ kiểm chứng C3 (durability).

## 3. Đóng góp cụ thể

### 3.1. Kiểm thử định dạng trace và bảo mật

Thêm [test_orchestrator_trace_security.py](../../src/tests/test_orchestrator_trace_security.py) với 17 trường hợp:

- Kiểm tra trace tồn tại, đọc được dưới dạng JSON object, có `run_id` trùng kết quả và `events` là một list.
- Kiểm tra trace được ghi trong lần chạy hiện tại; khi retry cùng claim, trace cũ phải được ghi mới.
- Bao phủ tám tình huống với seed `11` và `55`: refund thành công, thiếu mã đơn, đơn ngoài phạm vi principal, approval bị từ chối, quote bằng 0, claim chứa dữ liệu nhạy cảm ở parent/order, đơn không tồn tại và claim không tồn tại. Trường hợp thứ 17 kiểm tra ghi mới trace khi retry.
- Gắn các giá trị nhạy cảm giả, dễ nhận diện vào database tạm: số điện thoại, địa chỉ có Unicode, ghi chú nội bộ, tên hàng và token trong header. Kiểm tra cả giá trị bị lộ, không chỉ tên field.
- Quét `RunResult`, trace, các file phụ thuộc phạm vi `trace_files()` thu thập, cùng stdout/stderr. Đọc cả JSON đã giải mã để phát hiện PII bị biểu diễn bằng Unicode escape.
- Kiểm tra status tương ứng và chặn kết quả `not_implemented`, tránh coi một stub không xử lý nghiệp vụ là bằng chứng bảo mật đạt yêu cầu.

Test sử dụng database và thư mục tạm, không chỉnh sửa fixture nguồn trong `src/data/`. Các assertion không phụ thuộc tên event cụ thể, để người viết orchestrator có thể tổ chức trace theo cách riêng nhưng vẫn tuân contract.

### 3.2. Kiểm thử crash, resume và idempotency

Thêm [test_orchestrator_crash_resume.py](../../src/tests/test_orchestrator_crash_resume.py) với sáu trường hợp: ba điểm crash nhân hai seed `11`, `55`.

| Điểm crash | Refund đã commit ngay sau crash | Checkpoint `issue_refund` ngay sau crash |
| --- | --- | --- |
| `before_effect` | Chưa có | Chưa có |
| `after_effect_before_checkpoint` | Có một refund và một audit committed | Chưa có |
| `after_checkpoint` | Có một refund và một audit committed | Đã có |

Mỗi trường hợp sử dụng claim `C-012`, đơn `O-107`, mức hoàn 35.000 VND và thực hiện:

1. Chụp trạng thái ban đầu của ledger và audit.
2. Gọi Shape C với `crash_at`, yêu cầu phát sinh `InjectedCrash` đúng điểm đã chọn. Nếu shape bỏ qua tham số hoặc nuốt exception, test phải fail.
3. Mở connection mới để kiểm tra dữ liệu thực sự đã lưu sau crash.
4. Resume trên cùng database, không truyền crash injection.
5. Retry thêm một lần và kiểm tra trạng thái không thay đổi: đúng một refund 35.000 VND, một audit `refund_committed`, idempotency key `refund:C-012` và checkpoint hợp lệ.

Test đồng thời đối chiếu các dòng ledger/audit của claim khác với trạng thái ban đầu, nhằm phát hiện khoản hoàn phát sinh sai claim hoặc làm thay đổi các khoản hoàn đã có. Kết quả trả về của shape cũng được đối chiếu với dữ liệu đã commit.

### 3.3. Phối hợp và quyết định kỹ thuật

- Đánh dấu hai file bằng marker `student`, phù hợp bộ test tích hợp của lab. Cổng host mặc định tiếp tục kiểm tra phần nền; các test Shape C được chạy bằng lệnh riêng.
- Dùng allowlist và dữ liệu đã lọc làm ranh giới bảo mật. `Trace.add()` và `safe_dump()` không tự loại bỏ dữ liệu nhạy cảm; chỉ được ghi payload đã lọc. `make_handoff()` và các tool đọc là thành phần có sẵn cần sử dụng đúng.
- Kiểm tra ledger và audit qua connection mới thay vì chỉ tin status `resolved` trong kết quả.
- Phân biệt effect với checkpoint: ở cửa sổ `after_effect_before_checkpoint`, tiền đã commit dù checkpoint chưa có. Resume phải dựa vào idempotency của ledger để không hoàn thêm lần nữa.

## 4. Bằng chứng

Commit kỹ thuật cá nhân: [`488b585f6edaff2967f1600020ee7d82dbf26971`](https://github.com/SonVinUni/K5-L3A-DAY08-NewbieAI-FromWorkflowsToAgents/commit/488b585f6edaff2967f1600020ee7d82dbf26971), nội dung `test: cover orchestrator trace privacy and crash recovery`. Commit bổ sung đúng hai file test, đã push lên nhánh `dongtq`.

Các hàm test chính:

- `test_orchestrator_trace_is_current_valid_and_private`: 16 trường hợp tham số hóa.
- `test_orchestrator_retry_rewrites_stale_trace`: một trường hợp.
- `test_orchestrator_crash_then_resume_refunds_exactly_once`: sáu trường hợp tham số hóa.

Môi trường kiểm thử: Windows, Python 3.12.10, pytest 8.4.2, PyYAML 6.0.3. Các kết quả dưới đây được chạy bằng công cụ thực thi của Codex trong phiên làm việc ngày 16/11/2026, sau khi cập nhật triển khai Shape C tại commit `0f994eb`; code và test được lưu cùng nhau tại commit cá nhân `488b585`.

Từ thư mục `src/`, lệnh kiểm tra Shape C đã chạy:

```powershell
python -m pytest -q tests/test_orchestrator_trace_security.py tests/test_orchestrator_crash_resume.py tests/test_public_matrix.py -m student -o addopts= -k orchestrator --tb=short
```

Kết quả: **85 passed, 186 deselected**. Trong 85 trường hợp đạt có 62 trường hợp public của Shape C và 23 trường hợp thuộc hai file test cá nhân.

Lệnh kiểm tra cổng host đã chạy:

```powershell
python -m pytest -q
```

Kết quả sau khi tích hợp cả hai file test: **55 passed, 271 deselected**. Hai file mới thuộc marker `student` nên không được tính vào 55 test của cổng host.

Để chỉ chạy 23 trường hợp cá nhân:

```powershell
python -m pytest -q tests/test_orchestrator_trace_security.py tests/test_orchestrator_crash_resume.py -m student -o addopts=
```

Các số liệu trên là kết quả kiểm thử public và test bổ sung; không phải kết quả chấm hidden hoặc số liệu benchmark kiến trúc.

## 5. Công cụ AI đã dùng

Sử dụng OpenAI Codex để đọc contract và rubric, hỗ trợ viết hai bộ test, thực thi pytest, phân tích lỗi và hỗ trợ soạn báo cáo. Quá trình kiểm chứng dựa trên assertion, ledger/audit SQLite và output chạy lệnh thực tế.

Trước khi tích hợp code, test trace báo Shape C còn là stub và test crash báo không phát sinh `InjectedCrash`. Sau khi cập nhật code từ `main`, cùng các test này đạt trong lần chạy 85 trường hợp nêu trên. Điều này giúp phân biệt lỗi triển khai chưa hoàn thành với kết quả bảo mật đạt trên luồng xử lý thực tế.

AI được dùng trong quá trình phát triển và kiểm thử. Code lab vẫn sử dụng MockLLM theo quy định, không bổ sung lời gọi model thật hoặc API key. Kết quả hidden, benchmark và run id chưa được đo trong phần việc này không được suy diễn từ số test đã pass.

## 6. Tự đánh giá

Phần kiểm thử cá nhân đã hoàn thành với 23 trường hợp và được kiểm chứng sau khi tích hợp Shape C. Bộ test bổ sung kiểm tra trace ở nhiều nhánh kết thúc, sử dụng giá trị nhạy cảm riêng để phát hiện rò rỉ và kiểm tra cả ba cửa sổ crash bằng dữ liệu đã commit. Hai file test đã có commit kỹ thuật và được đẩy lên nhánh làm việc.

Giới hạn: các test bảo mật tìm những giá trị nhạy cảm đã biết trong dữ liệu thử, không chứng minh khả năng phát hiện mọi cách biến đổi hoặc mã hóa dữ liệu. Crash được mô phỏng bằng `InjectedCrash` và mở lại connection, chưa phải thử nghiệm tắt tiến trình hoặc mất điện thực tế. Bộ test chưa có kết quả hidden, chưa chứng minh an toàn trong mọi tình huống đồng thời và chưa thay thế benchmark/crash suite chính thức của nhóm.

Bài học kỹ thuật chính là trace thuộc ranh giới bảo mật và phải được ghi đúng ở mọi nhánh; đồng thời checkpoint không tự bảo đảm hoàn tiền đúng một lần. Việc giữ đúng một refund và một audit sau resume cần được chứng minh bằng ledger cùng cơ chế idempotency của host.
