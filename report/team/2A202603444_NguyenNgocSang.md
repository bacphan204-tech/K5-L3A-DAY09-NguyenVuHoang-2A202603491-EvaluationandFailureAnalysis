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
  - thêm `src/tests/test_eval_report_consistency.py` (7 test);
  - thêm `report/team/decision_adr.md`, `research_card.md`, `group_report.md`, `src/reports/ablation_r1.json`. Nội dung các file này trùng với bản đã có trên `main` ở commit `4ab0346`.
- **Nội dung 7 test:**
  - `benchmark.json` đúng quy cách nộp: đủ 4 shape, seed `11,22,33,44,55`, ≥ 10 claim, MockLLM, latency `unavailable`;
  - phần tóm tắt `by_shape` của `benchmark.json` và `ablation_r1.json` khớp đúng với `lab8.metrics.summarize` tính lại từ trial thô, nên bắt được nếu ai sửa tay JSON;
  - shape chọn, pass rate, pass^5, seed min–max, variance, cost/success trong ADR khớp `benchmark.json`;
  - mọi run id trích trong ADR có trong benchmark;
  - crash safety trong ADR khớp `crash.json` (20/20);
  - ablation R1 dùng cùng biến kiểm soát và cho cùng số liệu với benchmark chính.
- **Kết luận chính của phần đánh giá:**
  - Chọn `workflow`: cùng pass rate 1.000 với single và orchestrator, cost/success thấp nhất (27.86 VND mô phỏng), không gọi model.
  - Ablation R1: thêm investigator (workflow → orchestrator) không tăng pass rate, cost/success +33%.

<!-- Sang bổ sung: quyết định kỹ thuật bạn tự đưa ra, phần nào bạn tự viết/sửa. -->

## 4. Bằng chứng

- Commit `bd96f04`; merge `9c3b872`.
- `python -m pytest -q tests/test_eval_report_consistency.py` → 7 passed.
- `src/reports/benchmark.json`, `src/reports/ablation_r1.json`; run id dẫn chứng trong ADR, ví dụ `R-workflow-C-001-s11`, `R-swarm-C-002-s55`.

## 5. Công cụ AI đã dùng

<!-- Sang tự điền: công cụ nào, dùng vào việc gì, tự kiểm tra kết quả thế nào (RULES.md).
     Lưu ý: bản nháp test_eval_report_consistency.py và các báo cáo được soạn với Claude Code trong phiên của nhóm, cần ghi rõ ở đây. -->

...

## 6. Tự đánh giá

<!-- Sang tự viết. Không để AI viết phần này (RULES.md). -->

...
