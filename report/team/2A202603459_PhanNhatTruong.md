# Báo cáo cá nhân Lab 8 LogiX-VN

## 1. Thông tin thành viên

- Họ tên **Phan Nhật Trường**
- MSSV **2A202603459**
- Email `truong02052004@gmail.com`
- GitHub [pnt2504](https://github.com/pnt2504)

## 2. Phần việc phụ trách

Em phụ trách tích hợp phần ledger, policy và phục hồi sau crash cho ba luồng `workflow`, `single` và `orchestrator`. Em gộp hai nhánh đã triển khai workflow và single agent, hoàn thiện orchestrator rồi kiểm tra việc hoàn tiền khi chạy lại cùng một claim.

## 3. Đóng góp cụ thể

Trong `orchestrator.py`, orchestrator trả `failed` nếu không tìm thấy claim và trả `needs_info` nếu thiếu mã đơn. Với claim có mã đơn, orchestrator dùng `make_handoff` và chỉ cấp `lookup_order` vì investigator chỉ cần tra đơn trong phạm vi của chủ đơn. Handoff chỉ mang các trường trong allowlist, với `facts={}`. Trace của bước này lưu handoff đã lọc và trạng thái tra đơn. [Mã orchestrator](../../src/lab8/shapes/orchestrator.py)

Sau khi có dữ liệu đơn hợp lệ, orchestrator dùng `compute_compensation` để tính quote theo policy đang có hiệu lực. Với quote dương, nó gửi **toàn bộ quote** và `crash_at` vào `issue_refund`. Host kiểm tra approval có thuộc đúng claim hay không, trừ các khoản đã hoàn theo đơn và ghi ledger cùng audit trong một giao dịch. Ví dụ, ở T14 quote là 150.000 đồng. Host trừ 30.000 đồng đã hoàn trước đó và ghi khoản hoàn mới 120.000 đồng. [Hợp đồng kết quả](../../docs/contracts.md)

Trong workflow, em thêm sự kiện yêu cầu hoàn tiền trước khi gọi host để trace có dấu vết ngay cả khi có crash. `InjectedCrash` đi tới caller để bài kiểm tra có thể chạy lại trên cùng cơ sở dữ liệu. Payload tra cứu trong trace chỉ giữ kết quả cần cho việc kiểm tra, gồm trạng thái tìm thấy claim, quyền xem đơn và lý do. [Mã workflow](../../src/lab8/shapes/workflow.py)

Em bổ sung 9 ca kiểm thử cho ba luồng tại các điểm `before_effect`, `after_effect_before_checkpoint` và `after_checkpoint`. Mỗi ca kiểm ledger, audit và checkpoint ngay sau crash. Sau đó test chạy lại claim hai lần và xác nhận chỉ có một khoản hoàn 35.000 đồng, một audit `refund_committed` và một checkpoint cho bước hoàn tiền. [Test phục hồi](../../src/tests/test_shape_crash_recovery.py)

## 4. Bằng chứng

- Commit `77a65e2` gộp hai nhánh workflow và single agent. Phần triển khai và kiểm thử nằm trong `eaed81d`, đã được gộp vào `main` ở `8471efe`. [Commit triển khai](https://github.com/daothithuphuong-1801/K5-L3A-DAY08-PSTL-FromWorkflowsToAgents/commit/eaed81d59e93550003f2e3d56bcc772e0ce9d545)
- `python -m pytest -q -o addopts=` chạy từ `src/` cho kết quả **312 test pass** tại `eaed81d`. Lần kiểm tra riêng gồm **62 ca public của orchestrator và 9 ca phục hồi**, tất cả đều pass.
- `python -m lab8.crash_test --count 20 --out reports/crash.json` cho kết quả **20/20**. Kết quả chạy lại khớp với file đã commit. [Báo cáo crash](../../src/reports/crash.json)
- Khi thử với một file chứa dữ liệu sai định dạng SQLite, orchestrator phát sinh `sqlite3.DatabaseError`. Các ca phục hồi ở trên dùng cơ sở dữ liệu hợp lệ và mô phỏng crash tại bước hoàn tiền.

## 5. Công cụ AI đã dùng

Em dùng Codex để hỗ trợ tích hợp nhánh, triển khai orchestrator, viết test phục hồi và biên tập báo cáo. Codex đã chạy lại các lệnh ở mục 4 trong môi trường `src/.venv` và đối chiếu kết quả crash suite với `crash.json`. Việc kiểm tra approval thiếu hoặc thuộc claim khác cũng được thực hiện trên cả ba luồng bằng cách đối chiếu trạng thái `denied`, audit `policy_denied` và ledger SQLite. [Quy định sử dụng AI](../../RULES.md)

## 6. Tự đánh giá

Em đánh giá phần phục hồi dựa trên trạng thái thực tế của SQLite sau mỗi lần chạy. Ca quan trọng nhất là crash sau khi ledger đã ghi khoản hoàn nhưng trước khi lưu checkpoint. Nếu chỉ kiểm tra trạng thái `resolved` sau khi chạy lại, em có thể bỏ sót việc hoàn tiền hai lần. Vì vậy, test còn kiểm tra số dòng trong ledger và số audit `refund_committed`.

Phần em cần cải thiện là cách orchestrator xử lý lỗi đọc cơ sở dữ liệu. Với file SQLite sai định dạng, bước tiếp theo là xác định cách báo lỗi cho caller và bổ sung ca kiểm thử riêng để xác nhận luồng dừng trước khi yêu cầu hoàn tiền.
