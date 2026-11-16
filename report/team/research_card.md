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
| CÂU | R1 / R2 / R3 / R4 | R1 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Google Research 2026, *Towards a Science of Scaling Agent Systems*, Benchmark & Latency model (Slide 15, 24, 33) |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Multi-agent chỉ mang lại lợi ích về độ trễ khi task chia song song được; với luồng tuần tự, multi-agent giảm hiệu năng 39–70% |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Synthetic benchmark trên MockLLM, pricing cố định, DAG dependency đơn giản trên SQLite |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu chuyển các investigator đọc độc lập (order, policy) từ tuần tự sang song song, đường găng (critical path) sẽ rút ngắn ~38.9% theo định luật Amdahl mà pass rate vẫn giữ 1.000 |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Cố định: MockLLM, 14 public claims, seeds 11-55, pricing. Biến thay đổi duy nhất: Chế độ thực thi investigator (Tuần tự vs ThreadPoolExecutor song song) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: SQLite database lock khi nhiều thread cùng đọc. Đo bằng: Pass rate, Pass^5, Seed variance, Critical path speedup |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn):
  `python -m lab8.benchmark --shapes orchestrator,orchestrator_parallel --extra-shape orchestrator_parallel=bonus.orchestrator_parallel:run_claim --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/bonus_benchmark.json`
- Run id / file report: `src/reports/bonus_benchmark.json`
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu:
  ĐÚNG. Bản song song `orchestrator_parallel` đạt `pass_rate = 1.000`, `pass^5 = 1.000`, `variance = 0.000000` tương đương bản tuần tự, trong khi đường găng DAG rút ngắn từ 125ms xuống 90ms (Speedup đạt 1.389x, tức cải thiện 38.9% độ trễ theo Amdahl's Law). Không xảy ra lỗi database lock do mỗi worker thread mở kết nối SQLite độc lập.
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.
