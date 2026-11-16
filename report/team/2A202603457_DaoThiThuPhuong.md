# Báo cáo cá nhân — 2A202603457_DaoThiThuPhuong

## 1. Thông tin thành viên

- Họ tên: Đào Thị Thu Phương
- MSSV: 2A202603457
- Email: phuongdtt1824@gmail.com
- GitHub: [daothithuphuong-1801](https://github.com/daothithuphuong-1801)

## 2. Phần việc phụ trách

Integration: quản lý repo nhóm, ghép code của các thành viên về `main`, giữ cổng host (`python -m pytest -q`) xanh suốt quá trình, chạy benchmark và crash suite, và chốt bài nộp bằng `python -m lab8.check_submission`.

## 3. Đóng góp cụ thể

- **Cài đặt và kiểm tra môi trường (CP1):** Python 3.11.4, `.venv`, smoke pass, cổng host xanh, ma trận student lúc đầu 186 failed / 62 passed đúng như đề.
- **Ghép branch (CP2–CP3), commit `8471efe`:** `origin/main` bị force-push nên `main` local lệch hash dù cùng nội dung. Tôi cất thay đổi local vào stash, đưa `main` về `origin/main`, kiểm `git merge-base --is-ancestor` và `git merge-tree` để chắc không có conflict, rồi merge `NguyenVuHoang-…` (workflow), `feature/single-agent`, `codex/ledger-policy-crash`. Sau merge: cổng host xanh, ma trận student 248/248.
- **Tích hợp phần của Long:** Long gửi code dạng `feature-orchestrator.bundle`. Bản `orchestrator.py` của Long xung đột với bản của Trường đã có trong `main`. Nhóm giữ bản của Trường; tôi cherry-pick bộ test và báo cáo của Long (`edf344c`, `f31ac10`, giữ tác giả Long) và đánh dấu `xfail` 5 test chỉ đúng với thiết kế của Long (`452ff97`). Một trong 5 test đó phát hiện orchestrator hiện tại ném exception khi DB hỏng thay vì trả `failed`.
- **Benchmark (CP5), commit `3a7c959`:** chạy 4 shape × 5 seed trên 14 claim public, tạo `src/reports/benchmark.json`.
- **Ablation R1 và báo cáo nhóm, commit `4ab0346`:** chạy `workflow` vs `orchestrator` (`src/reports/ablation_r1.json`), điền `decision_adr.md`, `research_card.md`, `group_report.md`.
- **Test, commit `2509db5`:** thêm `src/tests/test_integration_shapes.py` (22 test):
  - 3 shape cho cùng `status`, `proposal_amount` và ledger trên cả 14 claim public;
  - mỗi shape giữ contract `run_claim` (run_id, status, trace sạch);
  - chạy cả 3 shape trên cùng một DB vẫn chỉ có một refund (idempotency của host dùng chung);
  - số tiền không đổi theo seed.
- **Chốt bài:** điền `TEAM.md` (`43f99dc`, `7baca51`), merge nhánh của Sang (`9c3b872`), báo cáo của Trường (`b718ac9`) và test của Tuấn Anh (`f500a41`, giữ bản báo cáo mới hơn của Tuấn Anh và file mẫu `MSSV_HoTen.md` mà nhánh đó đã xóa), ghi commit chốt vào ADR.

Quyết định kỹ thuật:

- Giữ orchestrator của Trường thay vì của Long để không phải chạy lại benchmark và viết lại số liệu, và vì bản đó đã qua 62/62 test student.
- Dùng `xfail` thay vì xóa test của Long, để giữ lại công của Long và để lộ rõ điểm yếu DB hỏng.

## 4. Bằng chứng

- Commit: `8471efe`, `3a7c959`, `4ab0346`, `452ff97`, `2509db5`, `43f99dc`, `9c3b872`, `7baca51`, `b718ac9`, `f500a41`.
- Test: `src/tests/test_integration_shapes.py` (22 test).
- Kết quả trên commit chốt, chạy trong `src/`:
  - `python -m pytest -q` → 156 passed, 5 xfailed
  - `python -m pytest -q -m student -o addopts=` → 248 passed
  - `python -m lab8.crash_test --count 20 --out reports/crash.json` → `crash_safety` 20/20
  - `python -m lab8.check_submission` → 0 FAIL, 0 WARN
- Report: `src/reports/benchmark.json`, `src/reports/ablation_r1.json`.

## 5. Công cụ AI đã dùng

- **Claude Code (trong VS Code):** đọc và tóm tắt đề, lên thứ tự công việc theo checkpoint; kiểm tra trạng thái branch, phát hiện force-push và nội dung bundle của Long; thực hiện merge; chạy benchmark, ablation và `check_submission`; soạn nháp ADR, research card, báo cáo nhóm, bảng đóng góp; viết `test_integration_shapes.py`.
- **Cách tôi tự kiểm tra:**
  - Mọi số liệu trong ADR và báo cáo nhóm lấy từ file JSON do `lab8.benchmark` / `lab8.crash_test` sinh ra, không chép tay; test `test_eval_report_consistency.py` kiểm số liệu và run id trong ADR khớp JSON.
  - Sau mỗi lần merge chạy lại cổng host và ma trận student.
  - Đối chiếu mô tả commit trong `TEAM.md` với `git show` của từng commit.
  - AI không được dùng để tạo số liệu benchmark, run id hay commit hash.

## 6. Tự đánh giá

<!-- Phương tự viết: điều làm tốt, điều chưa tốt, điều học được. Không để AI viết phần này (RULES.md). -->

...
