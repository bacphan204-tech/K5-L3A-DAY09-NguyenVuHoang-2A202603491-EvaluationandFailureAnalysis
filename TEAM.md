# Team

## Thông tin nhóm

- Tên nhóm: PSTL
- Lớp: L3A
- Repo nhóm: https://github.com/daothithuphuong-1801/K5-L3A-DAY08-PSTL-FromWorkflowsToAgents.git 
- URL repo, nhánh nộp, commit chốt: https://github.com/daothithuphuong-1801/K5-L3A-DAY08-PSTL-FromWorkflowsToAgents · nhánh `main` · commit chốt: `40cd3b8` (trùng `report/team/decision_adr.md`)

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Phạm Như Long | 2A202603395 |    plong20051901@gmail.com   | Long1806T | Orchestrator |
|   2 |    Đào Thị Thu Phương    | 2A202603457   | phuongdtt1824@gmail.com| daothithuphuong-1801       |   intergration      |
| 3 | Nguyễn Vũ Hoàng | 2A202603491 | bacphan204@gmail.com | bacphan204-tech | workflow |
| 4 | Đỗ Tuấn Anh | 2A202603465 | dotuananh30122004@gmail.com | tuancui0 | single-agent |
| 5 | Nguyễn Ngọc Sang | 2A202603444 | nguyenngocsang051003@gmail.com | ngocsangnguyen | eval/report| 
| 6 | Phan Nhật Trường | 2A202603459 | truong02052004@gmail.com | pnt2504 | ledger/policy/crash |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Phạm Như Long (2A202603395) | Orchestrator | `edf344c` (cherry-pick từ `b00d406` trên nhánh `feature/orchestrator`): bộ test shape C; `f31ac10` báo cáo cá nhân. Bản `orchestrator.py` riêng của Long không merge vào `main` (nhóm giữ bản `eaed81d`) | `src/tests/test_orchestrator_handoff.py` (20 test: 15 pass, 5 `xfail` vì viết cho thiết kế 2 investigator) |
| Đào Thị Thu Phương (2A202603457) | Integration | `8471efe` merge các nhánh shape vào `main`; `3a7c959` benchmark 4 shape × 5 seed; `4ab0346` ADR, research card, ablation R1, báo cáo nhóm; `452ff97` tích hợp test của Long | `src/tests/test_integration_shapes.py` (22 test, commit `2509db5`) |
| Nguyễn Vũ Hoàng (2A202603491) | Workflow | `53057be` cài shape A `workflow` | `src/tests/test_workflow.py` (1 test) |
| Đỗ Tuấn Anh (2A202603465) | Single agent | `90f7931` cài shape B `single` (ReAct ≤ 8 bước, model chỉ chọn tool đọc); `f5585eb` test shape single | `src/tests/test_single_agent.py` (48 test: prompt model không chứa ticket/secret, chỉ chọn tool đọc trong 8 bước, model không đổi được số tiền, chạm step cap thì fail-closed, trace sạch, thiếu `order_id` thì `needs_info`) |
| Nguyễn Ngọc Sang (2A202603444) | Eval/report | `bd96f04` (nhánh `ngocsangnguyen`, merge vào `main` ở `9c3b872`): test eval/report, ADR, research card, báo cáo nhóm, `reports/ablation_r1.json` | `src/tests/test_eval_report_consistency.py` (7 test: benchmark đúng quy cách, tóm tắt khớp trial thô, số liệu và run id trong ADR khớp file JSON) |
| Phan Nhật Trường (2A202603459) | Ledger/policy/crash | `eaed81d` cài shape C `orchestrator` đang dùng trong `main`; sửa workflow (không bắt `InjectedCrash`, bỏ `claim_id`/`order_id` khỏi trace); crash suite `reports/crash.json` 20/20 | `src/tests/test_shape_crash_recovery.py` (9 test) |

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
