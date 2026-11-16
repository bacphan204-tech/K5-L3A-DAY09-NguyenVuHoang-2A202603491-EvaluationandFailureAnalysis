# Báo cáo cá nhân — 2A202603459_PhanNhatTruong

## 1. Thông tin thành viên

- Họ tên: Phan Nhật Trường
- MSSV: 2A202603459
- Email: truong02052004@gmail.com
- GitHub: [pnt2504](https://github.com/pnt2504)

## 2. Phần việc phụ trách

Ledger/policy/crash: bảo đảm ghi tiền an toàn khi gọi lại và khi crash cho cả ba shape; hoàn thiện shape C `orchestrator` đang dùng trong `main`; chạy crash suite 20 điểm.

## 3. Đóng góp cụ thể

Commit chính: `eaed81d` "Complete refund orchestration and verify crash recovery" (nhánh `codex/ledger-policy-crash`, merge vào `main` ở `8471efe`).

- **Shape C `orchestrator`** (`src/lab8/shapes/orchestrator.py`):
  - orchestrator đọc claim qua `get_claim_view`; thiếu `order_id` → `needs_info`, claim không tồn tại → `failed`;
  - dựng handoff bằng `make_handoff` với `facts={}` và `allowed_tools=["lookup_order"]` (chỉ tool đọc), chạy qua `run_readonly_investigators`; trace ghi handoff đã sanitize, không ghi parent context;
  - orchestrator tự quote bằng `compute_compensation` và là bên duy nhất gọi `durability.issue_refund` với quote đầy đủ và `crash_at`; trace được `write()` trước khi gọi host.
- **Sửa workflow** (`src/lab8/shapes/workflow.py`):
  - bỏ import và xử lý `InjectedCrash` để crash đi tới caller (T12);
  - bỏ `claim_id` và `order_id` khỏi payload trace.
- **Crash suite:** `src/reports/crash.json` → `crash_safety` 20/20.
- **Test** `src/tests/test_shape_crash_recovery.py` (9 test = 3 shape × 3 điểm crash `before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`). Mỗi trường hợp kiểm:
  - crash thật sự xảy ra;
  - trạng thái ledger, audit, checkpoint đúng ngay sau crash;
  - resume và replay đều `resolved` 35 000 VND, ledger đúng một dòng, đúng một audit `refund_committed`.

<!-- Trường bổ sung: quyết định kỹ thuật bạn tự đưa ra (ví dụ vì sao chỉ giao lookup_order cho investigator). -->

## 4. Bằng chứng

- Commit `eaed81d`.
- `python -m pytest -q tests/test_shape_crash_recovery.py` → 9 passed.
- `python -m pytest -q -m student -o addopts= -k orchestrator` → 62 passed.
- `python -m lab8.crash_test --count 20 --out reports/crash.json` → `crash_safety` 20/20.
- Đã biết: test `test_unknown_claim_and_broken_store_return_failed_instead_of_raising` (của Long, đang `xfail`) cho thấy orchestrator ném `OperationalError` khi DB hỏng thay vì trả `failed`.

## 5. Công cụ AI đã dùng

<!-- Trường tự điền: công cụ nào, dùng vào việc gì, tự kiểm tra kết quả thế nào (RULES.md). Tên nhánh `codex/...` gợi ý có dùng Codex, cần ghi rõ. -->

...

## 6. Tự đánh giá

<!-- Trường tự viết. Không để AI viết phần này (RULES.md). -->

...
