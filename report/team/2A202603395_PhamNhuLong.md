# Báo cáo cá nhân — 2A202603395_PhamNhuLong

## 1. Thông tin thành viên

- Họ tên: Phạm Như Long
- MSSV: 2A202603395
- Email:
- GitHub: Long1806T

## 2. Phần việc phụ trách

Shape C `orchestrator` (`src/lab8/shapes/orchestrator.py`): orchestrator cộng các investigator chỉ đọc, handoff theo allowlist, trace an toàn. Kèm bộ test riêng cho shape C (`src/tests/test_orchestrator_handoff.py`).

## 3. Đóng góp cụ thể

**Commit chính:** `b00d406`, tiêu đề `feat(orchestrator): implement shape C with read-only investigators and allowlisted handoffs`, nằm trên nhánh `feature/orchestrator`.

Luồng xử lý:

```text
claim view -> plan -> investigator "order" -> investigator "policy"
           -> quote (code) -> host ghi refund (durability.issue_refund) -> soạn trả lời
```

Quyết định kỹ thuật:

- **Parent context chỉ là claim view.** Orchestrator đọc claim qua `get_claim_view`, nên không bao giờ cầm `parent_secret` hay raw header. MockLLM được đọc ticket để lập kế hoạch và soạn câu trả lời. Mã đơn, số ngày trễ và số tiền không lấy từ ticket.
- **Hai investigator, mỗi bên ít quyền nhất có thể.** Cả hai handoff đều tạo bằng `make_handoff`, tool đều là tool đọc:
  - `verify_order_scope`: `facts` rỗng, tool `lookup_order` + `lookup_refund_history`. Kiểm đơn có thuộc principal của claim không, lấy các field vận hành và số tiền đơn đã được hoàn.
  - `check_applicable_policy`: `facts` là các key trong `FACT_KEYS`, chép từ order view của investigator trước (không lấy từ ticket); tool `lookup_policy`, chỉ trả policy đang hiệu lực.
  - Đơn không thấy được (không tồn tại, hoặc của principal khác) thì `escalated` ngay, không gọi investigator policy.
- **Tiền do code tính, host ghi.** Orchestrator tính quote bằng `compute_compensation` trên order view của investigator, rồi đối chiếu các clause đã dùng với danh sách policy investigator thấy đang hiệu lực. Quote = 0 thì `denied` và không gọi host. Quote > 0 thì truyền **quote đầy đủ** vào `durability.issue_refund`. Host tự kiểm approval của chính claim, trừ phần các claim trước đã nhận trên cùng đơn, và giữ idempotency. Shape không tự trừ, nên T13 và T14 đúng.
- **Crash.** `crash_at` được truyền thẳng vào `issue_refund`. `InjectedCrash` không bị nuốt: orchestrator ghi event `crash` vào trace rồi raise lại (T12).
- **Không ném exception.** Lỗi khác (store hỏng, handoff bị từ chối) trả `failed` kèm trace hợp lệ. Trace chỉ ghi tên loại lỗi, không ghi message.
- **Trace.** Các event là `plan`, `handoff` (đã sanitize), `investigator` (field allowlist), `quote`, `host`, `decision` và `crash` khi có. Cố ý không ghi ticket, claim row, order row thô (SĐT, địa chỉ, ghi chú nội bộ, tên hàng), text của model và message exception.

Test đã thêm: `src/tests/test_orchestrator_handoff.py`, gồm 20 test, chạy trong cổng host:

- 12 claim public (T01–T10, T13, T14): pass `trial_pass`; mọi handoff chỉ có field allowlist, `facts` thuộc `FACT_KEYS`, tool chỉ đọc; trace và kết quả không chứa secret, `Bearer`, ticket hay PII của đơn.
- T02 ở seed 55: investigator policy nhận `delay_days = 2` từ order row, không lấy 5 từ ticket.
- Ticket injection có chứa canary: không ra tiền, không lọt vào trace.
- Trần theo đơn (T14): quote 150.000 VND được gửi đủ, host ghi 120.000 VND.
- Approval mượn của claim khác: `denied`, proposal là quote đầy đủ, có audit `policy_denied` lý do `approval_claim_mismatch`.
- `order_id` dùng chữ О Cyrillic (homoglyph): `escalated`, không khớp nhầm sang `O-107`.
- Gọi lại khi ledger đã có số khác: `failed`, `amount_conflict`, giữ dòng cũ.
- Crash `after_effect_before_checkpoint`: exception tới được caller, trace vẫn sạch, resume trả đúng một refund.
- Claim không tồn tại và store hỏng: trả `failed`, trace đúng contract, không ném exception.

## 4. Bằng chứng

Chạy trong `src/`. MockLLM cố định theo seed nên chạy lại ra cùng kết quả.

| Lệnh | Kết quả |
| --- | --- |
| `python -m pytest -q -m student -o addopts= -k orchestrator` | 62 passed |
| `python -m pytest -q` | 74 passed (54 test có sẵn + 20 test mới) |
| `python -m pytest -q tests/test_orchestrator_handoff.py` | 20 passed |
| `python -m lab8.benchmark --shapes orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out <file tạm>` | orchestrator (70 trial): pass rate 1.0, pass^5 1.0, seed min/max 1.0/1.0, variance 0, cost/success 68.83 VND (mô phỏng), mean steps 7.07. swarm: pass rate 0.986, pass^5 0.929, cost/success 49.57 VND |

Run id ví dụ:

- `R-orchestrator-C-014-s55`: `resolved`, proposal 150.000 VND, ledger 120.000 VND (4 model call, 7 tool call).
- `R-orchestrator-C-005-s55`: `escalated`, `outside_principal_scope`, trace không có SĐT, địa chỉ, tên hàng hay ghi chú nội bộ của `O-110`.

Ngày 16/11/2026 đã ghép thử nhánh này với `NguyenVuHoang-2A202603491` (workflow) và `feature/single-agent` (single). Kết quả: không có conflict, `python -m pytest -q` ra 75 passed, ma trận student ra 248 passed (4 shape × 62 test).

`src/reports/benchmark.json` chính thức sẽ được nhóm chạy lại khi đủ 4 shape.

## 5. Công cụ AI đã dùng

- **Claude (Anthropic):** đọc đề và contract, viết `orchestrator.py` và `test_orchestrator_handoff.py`, chạy các lệnh kiểm tra ở mục 4. Commit có dòng `Co-Authored-By: Claude`.
- **Cách kiểm tra:**
  - Chạy lại các lệnh ở mục 4.
  - Đọc trực tiếp file trace (ví dụ `traces/R-orchestrator-C-005-s55.json`) để chắc không có secret hay PII.
  - Thử cố ý làm hỏng orchestrator theo 5 cách: ghi ticket vào trace, tự trừ phần đã hoàn trước khi gọi host, nuốt `InjectedCrash`, lấy số ngày trễ từ ticket, không truyền `crash_at`. Cả 5 bản hỏng đều bị test mới bắt.

## 6. Tự đánh giá

_(Phần này tự viết, không dùng AI — xem [RULES.md](../../RULES.md).)_
