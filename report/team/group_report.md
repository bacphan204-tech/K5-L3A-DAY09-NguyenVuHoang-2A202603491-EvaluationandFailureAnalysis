# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: PSTL
- Thành viên: xem `TEAM.md`
- Commit chốt (trùng `report/team/decision_adr.md`): 3a7c959

## 2. Mục tiêu và cách tiếp cận

Bài toán: xử lý claim hoàn phí vận chuyển của LogiX-VN bằng bốn kiến trúc (A workflow, B single ReAct, C orchestrator, D swarm có sẵn), cùng chữ ký `run_claim`, rồi dùng số đo để chọn kiến trúc **ít tự chủ nhất mà vẫn làm được việc**. Nhóm hiểu ranh giới cốt lõi là: số tiền, approval, trần theo đơn và idempotency do **code và host** quyết; model chỉ được đọc và soạn chữ; chữ trong ticket là dữ liệu người dùng, không phải lệnh.

Chia việc (nhóm 6 người, theo gợi ý trong `TEAM.md`): integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report. Mỗi người làm trên branch riêng, integration merge về `main` sau khi cổng host (`python -m pytest -q`) xanh.

Thứ tự làm: (1) đọc `swarm_reference.py`, `business_rules.md`, `contracts.md` để nắm contract; (2) viết workflow trước vì là shape đơn giản và làm baseline; (3) single và orchestrator tái dùng cùng chuỗi bước đọc claim → đọc order → quote → gọi host; (4) thêm `crash_at` và test crash recovery; (5) chạy crash suite, benchmark, ablation và viết báo cáo.

## 3. Giải pháp

Cả ba shape đi cùng một chuỗi nghiệp vụ, chỉ khác **ai chọn bước tiếp theo**:

| Bước | Ai quyết | Cách làm |
| --- | --- | --- |
| Đọc claim | Code | `get_claim_view` (view allowlist, không `SELECT *`). Không tìm thấy → `failed`; thiếu `order_id` → `needs_info`, không ghi ledger |
| Đọc order | Code | `lookup_order(order_id, principal_id)`. Không thấy hoặc principal không sở hữu → `escalated`, không chép SĐT/địa chỉ/tên hàng/ghi chú |
| Tính tiền | Code | `compute_compensation` từ order row (cột `delay_days`, không đọc số ngày trong ticket), chỉ dùng policy hiện hành |
| Quote = 0 | Code | `denied`, `proposal_amount` 0, không gọi `issue_refund` |
| Quote > 0 | Host | `durability.issue_refund(conn, claim_id, <quote đầy đủ>, crash_at=crash_at)`. Host kiểm approval đúng claim, trừ phần đã hoàn của đơn (T13, T14), idempotency `refund:{claim_id}`. Host từ chối → `denied` (host ghi audit `policy_denied`); conflict → `failed`; thành công → `resolved` |
| Kết quả | Code | `lab8.shapes.common.finish()` tự lấy `refund_effects`, `committed_audit_count` từ ledger và ghi trace |

**A — workflow** (`src/lab8/shapes/workflow.py`): các bước trên viết cố định bằng Python, không gọi MockLLM (mean model calls = 0). Đây là shape ít tự chủ nhất.

**B — single** (`src/lab8/shapes/single.py`): một vòng `run_react` tối đa 8 bước. Hàm `decide` gọi MockLLM mỗi lượt (để cost tính đúng vào Budget) nhưng prompt chỉ chứa `claim_id` và tên tool trước đó, không chứa chữ trong ticket. `decide` chỉ được chọn tool đọc (`get_claim_view`, `lookup_order`, `compute_compensation`); vòng lặp trả quote ra ngoài, rồi **code của shape** mới gọi `issue_refund`. Model không chọn số tiền và không ghi refund.

**C — orchestrator** (`src/lab8/shapes/orchestrator.py`): orchestrator đọc claim, rồi dựng handoff bằng `make_handoff` với `facts={}` và `allowed_tools=["lookup_order"]` (chỉ tool đọc). Investigator chạy qua `run_readonly_investigators` (không có write registry). Orchestrator nhận order view, tự quote bằng code và là bên duy nhất gọi `issue_refund`. Trace ghi handoff đã sanitize (handoff.v1), không ghi parent context; `parent_secret` và header không bao giờ đi vào handoff.

**Durability**: cả ba shape truyền `crash_at` vào `issue_refund`, không bắt `InjectedCrash`, và gọi `trace.write()` ngay trước khi gọi host để crash vẫn để lại trace sạch. Resume trên cùng DB dựa vào ledger (idempotency key), không dựa vào checkpoint. Test thêm: `src/tests/test_workflow.py`, `src/tests/test_shape_crash_recovery.py`.

**Trace** ghi các event: `claim_lookup`, `decision`, `order_lookup`, `handoff` (chỉ orchestrator), `tool_result` / `agent_turn` (chỉ single), `quote`, `host_request`, `refund` / `host_result`. Cố ý không ghi claim row, order row thô, câu chữ ticket, `principal_id`, header.

## 4. Kết quả

Bộ public: 14 claim (T01–T14) × 5 seed `11,22,33,44,55` = 70 trial mỗi shape, MockLLM, `config/pricing.yaml`, SQLite mới mỗi trial. Nguồn: `src/reports/benchmark.json`.

| Shape | Pass rate | Pass^5 | Seed min–max | Variance | Cost/success (VND mô phỏng) | Mean steps | p95 steps | Mean model calls | Mean tool calls |
| --- | ---: | ---: | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| workflow | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 27.86 | 4.14 | 5 | 0.00 | 2.79 |
| single | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 30.39 | 8.21 | 9 | 3.79 | 2.79 |
| orchestrator | 1.000 | 1.000 | 1.000–1.000 | 0.000000 | 37.14 | 6.00 | 7 | 0.00 | 3.71 |
| swarm | 0.986 | 0.929 | 0.929–1.000 | 0.000816 | 49.57 | 2.80 | 3 | 3.00 | 4.66 |

- Trial fail duy nhất: `R-swarm-C-002-s55` (T02, seed 55) — `status resolved != denied`. Đây là điểm yếu cố ý của swarm: ở seed chia hết cho 5, swarm tin số ngày trong ticket ("trễ 5 ngày") thay vì cột `delay_days` (2 ngày). Ba shape của nhóm đọc order row nên pass trial này.
- Crash safety: **20/20** (`src/reports/crash.json`).
- Ma trận student: 248 passed / 0 failed trên cả 4 shape; cổng host `python -m pytest -q`: 64 passed.
- Ablation R1 (workflow vs orchestrator, `src/reports/ablation_r1.json`): pass rate không đổi, cost/success +33%, mean steps +1.86 — xem `research_card.md`.
- Tự kiểm trên `student_drop/`: chưa chạy (chờ giảng viên phát bộ hidden).

Quyết định kiến trúc và lý do: xem `report/team/decision_adr.md`. Phiếu nghiên cứu và ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

- **Windows không có lệnh `py`.** Hướng dẫn cài đặt dùng `py -3 -m venv .venv`, nhưng máy integration báo `py : The term 'py' is not recognized`. Xử lý: dùng `python -m venv .venv` (Python 3.11.4, trong khoảng cho phép); smoke in `lab8.smoke pass`.
- **`origin/main` bị force-push**, làm `main` local lệch hash với GitHub (ahead 1, behind 1) dù nội dung giống hệt (`git diff` rỗng). Xử lý: cất thay đổi local vào stash, đưa `main` về `origin/main`, rồi mới merge các branch. Nhóm thống nhất không force-push nữa (theo `RULES.md`).
- **Ghép branch của nhiều người.** Bốn branch (`NguyenVuHoang-…` workflow, `feature/single-agent`, `codex/ledger-policy-crash`, `Phạm-Như-Long_…` báo cáo) chồng nhau: branch ledger đã chứa sẵn branch workflow và single. Xử lý: kiểm `git merge-base --is-ancestor` và `git merge-tree` trước khi merge để chắc không có conflict; sau merge chạy lại cổng host (64 passed) và ma trận student (248 passed).
- **Trần theo đơn (T13, T14).** Dễ sai nếu shape tự trừ phần đã hoàn rồi mới gửi host (host trừ lần hai). Xử lý: luôn truyền quote đầy đủ; `proposal_amount` giữ quote, số ghi ledger do host tính.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp của từng thành viên nằm trong `TEAM.md` (bảng đóng góp) và báo cáo cá nhân `report/team/MSSV_HoTen.md`.

## 7. Hạn chế và hướng cải thiện

MockLLM không phải model sản xuất; latency là `unavailable`. Nêu thêm giới hạn của nhóm và điều kiện làm kết luận thay đổi.

- Bộ public chỉ có 14 claim và cả ba shape của nhóm đã bão hòa ở 1.000, nên benchmark public không phân biệt được độ đúng giữa A, B, C; chỉ phân biệt được cost và số bước. Bộ hidden (40 claim, có ticket nói dối, injection, homoglyph, principal lẫn) mới là phép thử thật.
- Cost là VND mô phỏng; với model thật, tỷ lệ cost giữa model call và tool call có thể khác, làm thứ tự cost/success thay đổi.
- Workflow phụ thuộc hoàn toàn vào dữ liệu có cấu trúc. Nếu xuất hiện loại claim cần hiểu ngôn ngữ tự nhiên (ticket mơ hồ, cần hỏi lại khách) hoặc nhiều nhánh điều tra độc lập chạy song song được, kết luận chọn workflow có thể đảo lại.
- Hướng cải thiện: chạy tự kiểm trên `student_drop/` khi có; thêm test đối kháng (ticket nói quá số ngày, approval của claim khác, homoglyph trong `order_id`).

## 8. Tài liệu tham khảo

- Đề bài và tài liệu lab: `README.md`, `docs/business_rules.md`, `docs/contracts.md`, `docs/metrics.md`, `docs/architecture.md`.
- Google Research 2026, *Towards a Science of Scaling Agent Systems* (thư viện tham khảo slide 68–69) — dùng cho research card R1.
- Code tham chiếu: `src/lab8/shapes/swarm_reference.py`.
