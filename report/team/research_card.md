# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R1 · Scaling |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Google Research 2026, *Towards a Science of Scaling Agent Systems* (thư viện tham khảo slide 68–69; bài lý thuyết N8 slide 15, 24, 33, 41). Loại bằng chứng: benchmark có kiểm soát trên nhiều kiến trúc agent |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Multi-agent chỉ có lợi khi task chia song song được; với task tuần tự, các biến thể multi-agent trong nghiên cứu làm hiệu năng giảm 39–70% |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Kết luận đo trên LLM thật và các benchmark task của nghiên cứu, không phải claims hoàn phí; con số 39–70% chỉ đúng cho các cấu hình và task trong paper. Lab dùng MockLLM cố định theo seed, 14 claim public, cost mô phỏng, latency `unavailable` |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Claims flow là tuần tự (claim → order → quote → host; quote phụ thuộc order). Thêm investigator (workflow A → orchestrator C) sẽ **không tăng** pass rate (giữ 1.000, chênh 0) nhưng **tăng** cost/success khoảng +30% và mean steps khoảng +2, vì thêm một vòng handoff + tool call mà không có nhánh nào chạy song song |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, tool manifest v1.1, seed `11,22,33,44,55`, corpus `data/public_claims.jsonl` (14 claim), `config/pricing.yaml`, evaluator `lab8.benchmark`, SQLite mới mỗi trial, `as_of` 2026-11-15. Biến duy nhất được thay đổi: topology — có hay không có lớp investigator read-only qua handoff (`workflow` vs `orchestrator`) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: chi phí điều phối (coordination overhead) — thêm agent mà không thêm độ đúng. Đo bằng cost/success và mean/p95 steps với pass rate, pass^5, variance giữ nguyên; đồng thời kiểm handoff không làm lộ secret (leak = 0, qua `test_handoff_secret.py` và trace scan của benchmark) |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,orchestrator --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/ablation_r1.json`
- Run id / file report: `src/reports/ablation_r1.json`; ví dụ cặp run id cùng claim, cùng seed: `R-workflow-C-014-s55` vs `R-orchestrator-C-014-s55`. Cùng số liệu có trong `src/reports/benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: **Đúng.** Pass rate 1.000 → 1.000, pass^5 1.000 → 1.000, variance 0 → 0 (không đổi). Cost/success 27.86 → 37.14 VND mô phỏng (+33%). Mean steps 4.14 → 6.00 (+1.86), p95 steps 5 → 7. Mean tool calls 2.79 → 3.71 (+1 tool call ở các claim có order). Model calls 0 → 0, nên toàn bộ chênh cost đến từ tool call của lớp investigator, phù hợp với nhận định của nguồn rằng task tuần tự không hưởng lợi từ thêm agent
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`. Vì latency không đo được, ablation chỉ cho thấy overhead về cost và số bước, không cho biết độ trễ thực. Pass rate đã bão hòa ở 1.000 trên 14 claim public nên không thể quan sát mức giảm hiệu năng 39–70% như trong paper; cần bộ claim khó hơn (hidden) hoặc model thật để kiểm tra phần đó.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
