# Team

## Thông tin nhóm

- Tên nhóm:
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-TenNhom-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Như Long | 2A202603395 |       | Long1806T | Orchestrator + handoff (shape C) |
|   2 |        |      |       |        |         |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phạm Như Long (2A202603395) | Orchestrator + handoff (shape C) | `b00d406` feat(orchestrator): implement shape C with read-only investigators and allowlisted handoffs (nhánh `feature/orchestrator`) | `src/tests/test_orchestrator_handoff.py` (20 test mới) |

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

## 2A202603395_PhamNhuLong

### Phần việc phụ trách

- Shape C `orchestrator` (`src/lab8/shapes/orchestrator.py`): orchestrator, handoff allowlist, investigator chỉ có tool đọc, trace an toàn.

### Đóng góp cụ thể

- Viết shape C: orchestrator đọc claim qua `get_claim_view`, giao việc cho 2 investigator chỉ đọc bằng `make_handoff` (`lookup_order` + `lookup_refund_history`; `lookup_policy`), tự tính quote bằng `compute_compensation`, truyền quote đầy đủ và `crash_at` vào `durability.issue_refund`.
- Thêm `src/tests/test_orchestrator_handoff.py` (20 test): handoff allowlist, trace không lộ secret/PII/ticket, ticket injection, trần theo đơn, approval mượn, homoglyph, amount conflict, crash rồi resume, không ném exception.

### Bằng chứng

- Commit `b00d406` (`feat(orchestrator): implement shape C with read-only investigators and allowlisted handoffs`) trên nhánh `feature/orchestrator`.
- `python -m pytest -q -m student -o addopts= -k orchestrator` → 62 passed; `python -m pytest -q` → 74 passed.
- Báo cáo cá nhân: [`report/team/2A202603395_PhamNhuLong.md`](report/team/2A202603395_PhamNhuLong.md).
