# Báo cáo cá nhân — Võ Sỹ Thái Sơn

## 1. Thông tin thành viên

- Họ tên: Võ Sỹ Thái Sơn
- MSSV: 2A202603623
- Email: vosythaison@gmail.com
- GitHub: SonVinUni

## 2. Phần việc phụ trách

Tôi phụ trách Shape A — deterministic workflow. Mục tiêu của phần này là xây dựng một luồng xử lý cố định bằng Python: đọc claim, kiểm tra order, tính quote từ dữ liệu có cấu trúc, rồi chuyển quote đầy đủ cho host để kiểm approval, giới hạn hoàn tiền và ghi ledger.

## 3. Đóng góp cụ thể

Tôi đã triển khai `src/lab8/shapes/workflow.py` và giữ nguyên chữ ký `run_claim`. Luồng xử lý sử dụng `get_claim_view` để đọc các field claim được phép dùng, xử lý riêng trường hợp thiếu `order_id`, dùng `lookup_order` để kiểm tra order có thuộc đúng principal hay không, và chuyển các trường hợp không nhìn thấy order sang trạng thái `escalated`.

Phần tính tiền không đọc số ngày trong ticket và không để model quyết định số tiền. Quote được tính bằng `compute_compensation` từ order và policy hiện hành. Khi quote bằng 0, workflow trả về `denied` mà không gọi cổng ghi refund. Khi quote lớn hơn 0, workflow truyền toàn bộ quote vào `durability.issue_refund`; workflow không tự trừ các khoản refund trước đó vì việc giới hạn theo order thuộc về host.

Tôi cũng xử lý các nhánh kết quả bằng `finish`, nhờ đó `RunResult` tự đối chiếu effect trong ledger, số audit committed và trace. Trace chỉ ghi các field cần thiết, không ghi ticket nguyên văn, secret, raw headers, số điện thoại, địa chỉ, tên hàng hoặc ghi chú nội bộ.

Để kiểm tra quy tắc approval, tôi thêm test `src/tests/test_workflow_policy_denied.py`. Test này dùng claim C-007 để kiểm tra trường hợp quote vẫn dương nhưng approval bị từ chối: không có dòng refund, không có audit `refund_committed`, và có audit `policy_denied`.

Commit chính của phần này là `fff3326` với message `Implement deterministic workflow refund shape`. Phần workflow sau đó được merge vào `main` trong commit merge `dba7131`.

## 4. Bằng chứng

Các lệnh kiểm tra đã chạy trong thư mục `src/`:

```text
python -m lab8.smoke
python -m pytest -q -m student -o addopts= -k workflow
python -m pytest -q
python -m lab8.crash_test --count 20 --out reports/crash.json
```

Kết quả ghi nhận:

```text
workflow student tests: 62 passed
host tests: 54 passed ở thời điểm commit workflow
crash safety: 20/20
```

Một lần chạy thử claim C-001 cho kết quả `resolved`, `proposal_amount=35000`, một refund effect 35000 VND và một audit `refund_committed`.

## 5. Công cụ AI đã dùng

Tôi có sử dụng Claude như một công cụ hỗ trợ trong quá trình làm bài. Claude được dùng để giải thích README và contract, gợi ý cách nối các hàm có sẵn, rà soát trace và các nhánh crash, cũng như hỗ trợ đọc kết quả pytest. Tôi không dùng AI để bịa số liệu benchmark hoặc tự khai kết quả. Sau khi nhận gợi ý, tôi kiểm tra lại bằng cách đọc code, chạy test trên máy và đối chiếu kết quả với các contract trong `docs/`.

## 6. Tự đánh giá

Phần tôi hiểu rõ nhất sau khi làm là ranh giới giữa shape và host. Shape chịu trách nhiệm điều phối các bước đọc và tạo quote; còn approval, giới hạn hoàn tiền theo order, idempotency và ghi ledger phải do host kiểm soát. Việc truyền quote đầy đủ vào `issue_refund` là điểm quan trọng, vì nếu workflow tự trừ refund cũ thì host sẽ trừ lần nữa và làm sai các trường hợp như T14.

Phần cần tiếp tục theo dõi là kiểm tra toàn bộ report và benchmark của nhóm sau khi các shape khác được merge. Những số liệu cuối cùng nên lấy trực tiếp từ các file report được sinh bằng lệnh benchmark, không ghi thủ công từ một lần chạy riêng lẻ.
