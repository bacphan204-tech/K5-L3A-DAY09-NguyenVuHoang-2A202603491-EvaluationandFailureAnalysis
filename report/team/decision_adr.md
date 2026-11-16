# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 3a7c959
- Run id dẫn chứng: R-workflow-C-001-s11, R-workflow-C-013-s55, R-workflow-C-014-s55 (so sánh: R-swarm-C-002-s55 là trial fail duy nhất của swarm) — src/reports/benchmark.json
- Pass rate / pass^5 / seed min-max / variance: 1.000 / 1.000 / 1.000–1.000 / 0.000000 (14 claim public × 5 seed = 70 trial, src/reports/benchmark.json)
- Cost per success (mô phỏng, VND): 27.86 (tổng 1950 VND mô phỏng / 70 trial pass)
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0 trên bộ public (70/70 trial workflow pass; một trial chỉ pass khi trace sạch chuỗi cấm và ledger khớp nhãn). Số chính thức trên hidden do grader của giảng viên chấm

Ví dụ một dòng đã điền: `- Shape chọn: workflow`, `- Crash safety: 20/20 (src/reports/crash.json)`.

## Vì sao shape này

Trên cùng corpus (14 claim public T01–T14), cùng 5 seed `11,22,33,44,55`, cùng MockLLM, cùng `config/pricing.yaml` và cùng evaluator (`src/reports/benchmark.json`), ba shape nhóm viết đều đạt pass rate 1.000, pass^5 1.000, variance 0. Swarm đạt 0.986 / 0.929 vì fail T02 ở seed 55 (`R-swarm-C-002-s55`: trả `resolved` thay vì `denied` do tin số ngày trong ticket). Khi độ đúng ngang nhau, nhóm chọn shape **ít tự chủ nhất và rẻ nhất**: workflow có cost/success 27.86 VND mô phỏng, so với 30.39 (single), 37.14 (orchestrator), 49.57 (swarm); workflow cũng không gọi model (mean model calls = 0) nên không có bước nào mà MockLLM có thể ảnh hưởng tới quyết định tiền. Mọi quyết định (needs_info, escalated, quote, gọi host) là code cố định, dễ đọc và dễ kiểm toán.

## Có đáng đổi lấy multi-agent không

Không, với dữ liệu hiện có. Orchestrator không tăng pass rate (cùng 1.000) nhưng tăng cost/success 33% (27.86 → 37.14) và mean steps từ 4.14 lên 6.00, vì thêm một vòng handoff + investigator cho bước `lookup_order` vốn tuần tự (quote phụ thuộc order). Swarm vừa đắt nhất vừa có variance khác 0. Nếu một shape multi-agent pass nhỉnh hơn mà cost và variance cao hơn, nhóm vẫn giữ workflow trừ khi phần chênh nằm ở các claim mà luật cố định không xử lý được. Quyết định sẽ đảo lại khi: (1) bộ hidden có loại claim mà workflow fail có hệ thống vì cần hiểu ngôn ngữ tự nhiên (ticket mơ hồ, cần hỏi thêm) chứ không chỉ đọc cột có cấu trúc; (2) task có nhiều nhánh điều tra độc lập chạy song song được, để investigator rút ngắn critical path; (3) dùng model thật mà chi phí gọi model thấp hơn đáng kể so với giá trị sửa sai.

## Trace và bằng chứng

Trace workflow (`lab8.trace.Trace`, file `traces/<run_id>.json`) ghi các event: `claim_lookup` (chỉ `found`), `decision` (status + lý do khi `needs_info`), `order_lookup` (`visible`, `reason`), `quote` (`amount_vnd`, `reason`, `clause_ids`, `parts`), `host_request` (`issue_refund`, `proposal_amount`), `refund` (`ok`, `reason`, `conflict`). Ví dụ `R-workflow-C-001-s11`: claim_lookup → order_lookup → quote 35000 (SHIP-LATE-03) → host_request → refund committed.

Cố ý **không** ghi: claim row và order row thô (claim row có `parent_secret`, order row có SĐT, địa chỉ, tên hàng, ghi chú nội bộ); câu chữ ticket (dữ liệu người dùng, có thể chứa injection hoặc PII); `principal_id` và header. Trace chỉ chứa field đã qua allowlist của `get_claim_view` / `lookup_order` và kết quả quote. Trace được `write()` trước khi gọi `issue_refund`, nên crash vẫn để lại bằng chứng sạch.

Nếu trace nằm ngoài chỗ scanner quét được, đội security không thể chứng minh hệ thống sạch. Vì vậy nhóm giữ mọi output của shape trong `traces/<run_id>*`, không in dữ liệu claim ra stdout, và dựa vào bằng chứng có thể chạy lại: test `test_handoff_secret.py`, `test_trace_contract.py`, ma trận student 248/248 pass, và benchmark 70/70 trial workflow pass (mỗi trial pass bao gồm quét chuỗi cấm).

## Giới hạn

MockLLM không phải model sản xuất. Latency để `unavailable`. Không sửa JSON cho đủ 0.75.

Bộ public chỉ có 14 claim; pass rate 1.000 trên public không đảm bảo pass rate trên 40 hidden claim. Cost là VND mô phỏng theo `config/pricing.yaml`, không phải hóa đơn API. Workflow không gọi model nên chênh cost chủ yếu đến từ số tool call; với model thật, khoảng cách cost giữa các shape có thể khác.
