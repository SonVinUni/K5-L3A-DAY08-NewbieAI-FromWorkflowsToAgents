# Team

## Thông tin nhóm

- Tên nhóm: NewbieAI
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-NewbieAI-FromWorkflowsToAgents`
- URL repo, nhánh nộp, commit chốt: https://github.com/SonVinUni/K5-L3A-DAY08-NewbieAI-FromWorkflowsToAgents.git; nhánh `main`; commit hiện tại `43a140b`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Võ Sỹ Thái Sơn | 2A202603623 | vosythaison@gmail.com | SonVinUni | Shape A (workflow) + integration, báo cáo chung |
|   2 | Trần Nguyễn Anh Phong | 2A202603629 | anhphong7cpct@gmail.com | anhphongtrng | Shape B (Single Agent) + ReAct, Budget, Testing |
|   3 |   Trần Xuân Trường     | 2A202603617      |  truongxuan2834@gmail.com     |   XuanTruong28     |   Shape C (orchestrator) + core flow & durability and Bonus part Topic B1      |
|   4 |    Trần Quốc Đông    | 2A202603643     | dongtranquoc951@gmail.com      |   dongtranquoc951     |  Shape C (orchestrator) + handoff & security trace, testing       |



Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Võ Sỹ Thái Sơn | Shape A (workflow) + integration, báo cáo chung | fff3326 (Implement deterministic workflow refund shape) | src/tests/test_workflow_policy_denied.py |
| Trần Nguyễn Anh Phong | Shape B (Single Agent) + ReAct, Budget, Testing | test(single-agent): add custom tests for claim processing and idempotency | src/tests/test_single_agent_custom.py  |
| Trần Xuân Trường | Shape C (orchestrator) + core flow & durability, bonus B1 | 0f994eb (hàm orchestrator và test của thành viên 3 (mục C ) trong team) | src/tests/test_n3_orchestrator.py |
| Trần Quốc Đông | Shape C (orchestrator) + handoff & security trace, testing | 5397a3c (test: cover orchestrator trace privacy and crash recovery) | src/tests/test_orchestrator_trace_security.py, src/tests/test_orchestrator_crash_resume.py |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```

Ví dụ: `report/team/20241234_NguyenVanAn.md`.

Sao chép mục dưới đây cho từng thành viên.

## MSSV_HoTen

### Phần việc phụ trách

-

### Đóng góp cụ thể

-

### Bằng chứng

-
