# Báo cáo cá nhân — 2A202603444_NguyenNgocSang

## 1. Thông tin thành viên

- Họ tên: Nguyễn Ngọc Sang
- MSSV: 2A202603444
- Email: nguyenngocsang051003@gmail.com
- GitHub: [ngocsangnguyen](https://github.com/ngocsangnguyen)

## 2. Phần việc phụ trách

Eval/report: kiểm tra số liệu benchmark, quyết định kiến trúc (ADR), research card và ablation, báo cáo nhóm; viết test bảo đảm số liệu trong báo cáo khớp với file do công cụ sinh ra.

## 3. Đóng góp cụ thể

- **Commit `bd96f04`** (nhánh `ngocsangnguyen`, merge vào `main` ở `9c3b872`):
  - Thêm `src/tests/test_eval_report_consistency.py` (7 test).
  - Thêm `report/team/decision_adr.md`, `research_card.md`, `group_report.md` và `src/reports/ablation_r1.json`. Nội dung các file này trùng với bản đã có trên `main` ở commit `4ab0346`.
- **Nội dung 7 test:**
  - Kiểm tra `benchmark.json` đúng quy cách nộp: đủ 4 shape, seed `11,22,33,44,55`, ít nhất 10 claim, dùng MockLLM và latency là `unavailable`.
  - Kiểm tra phần tóm tắt `by_shape` trong `benchmark.json` và `ablation_r1.json` khớp với kết quả `lab8.metrics.summarize` tính lại từ trial thô, giúp phát hiện JSON bị sửa tay.
  - Kiểm tra shape được chọn, pass rate, pass^5, seed min–max, variance và cost/success trong ADR khớp với `benchmark.json`.
  - Kiểm tra mọi run ID được dẫn trong ADR đều có trong benchmark.
  - Kiểm tra crash safety trong ADR khớp với `crash.json` (20/20).
  - Kiểm tra ablation R1 giữ cùng biến kiểm soát và cho số liệu khớp với benchmark chính.
- **Kết luận chính của phần đánh giá:**
  - Chọn `workflow`: có cùng pass rate 1.000 với `single` và `orchestrator`, cost/success thấp nhất (27.86 VND mô phỏng) và không gọi model.
  - Ablation R1 cho thấy thêm investigator (`workflow` → `orchestrator`) không làm tăng pass rate, trong khi cost/success tăng 33%.

Quyết định kỹ thuật được ghi nhận trong ADR là chọn `workflow`: kết quả có pass rate ngang `single` và `orchestrator`, nhưng cost/success thấp nhất và không gọi model. Phần việc trong commit của tôi tập trung vào bộ test đối chiếu số liệu benchmark/ablation với dữ liệu thô và các báo cáo liên quan.

## 4. Bằng chứng

- Commit `bd96f04`; merge `9c3b872`.
- Theo kết quả ghi nhận khi tạo commit: `python -m pytest -q tests/test_eval_report_consistency.py` → 7 passed.
- Chạy lại trong workspace hiện tại: `python -m pytest -q src/tests/test_eval_report_consistency.py` → 1 passed, 6 skipped vì chưa có `benchmark.json` và `crash.json` được sinh ra.
- Báo cáo: `src/reports/benchmark.json` và `src/reports/ablation_r1.json`; run ID dẫn chứng trong ADR, ví dụ `R-workflow-C-001-s11` và `R-swarm-C-002-s55`.

## 5. Công cụ AI đã dùng

- **Claude Code:** hỗ trợ soạn bản nháp `test_eval_report_consistency.py` và các báo cáo của nhóm.
- **Copilot trong VS Code:** hỗ trợ rà soát thông tin và chỉnh sửa cách trình bày báo cáo cá nhân.
- **Kiểm chứng:** đối chiếu số liệu qua bộ test nhất quán báo cáo. Lần chạy được ghi nhận khi tạo commit có 7 test đạt; lần chạy lại trong workspace hiện tại có 1 test đạt, 6 test bị bỏ qua do thiếu `benchmark.json` và `crash.json`.

## 6. Tự đánh giá

Tôi đã hoàn thành phần eval/report được giao, tập trung vào việc kiểm tra tính nhất quán giữa số liệu benchmark, ablation, ADR và các file kết quả; các test giúp phát hiện trường hợp số tổng hợp không khớp với dữ liệu trial thô. Qua công việc này, tôi hiểu rõ hơn cách dùng bằng chứng định lượng để so sánh các shape và giải thích lựa chọn kiến trúc, thay vì chỉ dựa vào nhận định chủ quan.

Điểm tôi cần cải thiện là bảo đảm các file kết quả cần thiết có sẵn khi chạy lại toàn bộ test. Trong workspace hiện tại, 6 test bị bỏ qua do thiếu `benchmark.json` và `crash.json`, nên tôi chưa xác nhận lại được toàn bộ bộ kiểm tra ở môi trường này. Lần sau, tôi sẽ kiểm tra quy trình sinh dữ liệu và chạy lại test sau khi tạo đủ báo cáo trước khi chốt phần đánh giá.
