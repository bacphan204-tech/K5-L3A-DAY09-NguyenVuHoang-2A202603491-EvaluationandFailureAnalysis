# Báo cáo cá nhân — Nguyễn Vũ Hoàng

## 1. Thông tin thành viên

- Họ tên: Nguyễn Vũ Hoàng
- MSSV: 2A202603491
- Email: 26ai.hoangnv11@vinuni.edu.vn
- GitHub: [bacphan204-tech](https://github.com/bacphan204-tech)

## 2. Phần việc phụ trách

Em phụ trách Shape A — `workflow`: xây dựng luồng xử lý claim theo các bước Python cố định, dùng dữ liệu order có cấu trúc để tính quote và gọi host khi cần hoàn tiền.

## 3. Đóng góp cụ thể

- Triển khai `run_claim` trong [`src/lab8/shapes/workflow.py`](../../src/lab8/shapes/workflow.py): xử lý các trường hợp thiếu mã đơn, không tìm thấy hoặc không thuộc quyền truy cập, không đủ điều kiện hoàn và đủ điều kiện hoàn.
- Dùng `compute_compensation` trên thông tin order thay vì tin số ngày ghi trong ticket. Khi quote dương, chuyển nguyên quote và `crash_at` cho host để host kiểm tra approval, giới hạn hoàn theo đơn và xử lý retry.
- Ghi trace cho các nhánh kết quả và đóng kết nối database sau khi chạy.
- Thêm [`src/tests/test_workflow.py`](../../src/tests/test_workflow.py) để kiểm tra claim không tồn tại trả về `failed` và có trace hợp lệ.
- Commit chính: [`53057bedf778586d8522521a6d0500df779c8947`](https://github.com/daothithuphuong-1801/K5-L3A-DAY08-PSTL-FromWorkflowsToAgents/commit/53057bedf778586d8522521a6d0500df779c8947).

## 4. Bằng chứng

Đã chạy các lệnh sau trong thư mục `src/`:

```powershell
.\.venv\Scripts\python.exe -m pytest -q --basetemp=.pytest-tmp tests/test_workflow.py
.\.venv\Scripts\python.exe -m pytest -q --basetemp=.pytest-tmp -m student -o addopts= -k workflow
.\.venv\Scripts\python.exe -m pytest -q --basetemp=.pytest-tmp
```

Kết quả lần lượt: **1 passed**, **62 passed** cho workflow, và **55 passed** ở host suite (248 test student được bỏ qua). Commit workflow đã được push lên nhánh `NguyenVuHoang-2A202603491`.

## 5. Công cụ AI đã dùng

Em có dùng Copilot/AI assistant để hỗ trợ đọc yêu cầu, triển khai workflow và viết test. Em tự kiểm tra kết quả bằng cách chạy test mới, ma trận student của workflow và host suite; đồng thời kiểm tra commit đã có trên nhánh GitHub.

## 6. Tự đánh giá

Phần workflow của em đã chạy qua các test được giao. Qua phần này em hiểu rõ hơn vì sao phải dùng thông tin order thay vì tin nội dung ticket, và vì sao thao tác ghi tiền nên để host kiểm tra. Em sẽ cần chạy lại toàn bộ test student và xem kết quả benchmark khi nhóm ghép xong các shape còn lại.
