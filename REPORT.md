# Báo cáo LAB Day 26: COLOSSEUM · Đấu Trường Agent

## Thông tin cá nhân
- Họ Tên: Nguyễn Tuấn Phong
- MSSV: 2A202601038
- Khoá: K3B
- Lớp: E403

## Kế hoạch LAB Day 26

### Tasks chính
| Task |  Mô tả  | Trạng thái |
|------|---------|------------|
| 0    | Setup Môi trường & Tải World (Corpus) | ✅ PASS |
| 1    | ATTACK (deck/): Soạn ASK + mutation manifest | ✅ PASS |
| 2    | DEFEND (agent/): Xây dựng agent xử lý MCP/A2A có guardrail | ✅ PASS |
| 3    | PROSECUTE (eval/): Xây dựng logic luận tội (eval trace) | ✅ PASS |

### Ghi chú tiến độ (Log)
- **Task 0 (Setup & World Corpus)**:
  - Cài đặt môi trường `python 3.12` venv và `pytest`.
  - Tải thành công `world-df8c55dabb35` (24,750 pages) từ repo gốc.
  - `make doctor` PASS: `0 failing check(s)`, sẵn sàng đấu (spar).
- **Task 1 (ATTACK - deck/)**:
  - Cấu hình 14 lá bài (10 attacks + 4 blanks) bao phủ 3 tầng (MCP: 4, Gateway: 2, A2A: 4) và 9/9 classes.
  - Chuẩn hóa `defense_event` về `gateway.denied`.
  - `make validate` PASS toàn bộ các ràng buộc luật thi đấu.
- **Task 2 (DEFEND - agent/)**:
  - Xây dựng hệ thống điều khiển Gateway 4 tầng (`ROUTE`, `ADMIT`, `AUTHORIZE`, `BUDGET`):
    - 4 lớp kiểm tra định danh Agent Card, công bố kỹ năng, Audience token `aud` và Act target ownership (`target == ctx.act`).
    - Phòng thủ ghi an toàn (bắt buộc `If-Match` etag, `Idempotency-Key` đơn nhất, kiểm tra scope `wiki.write`).
    - Xác thực Lease thời gian thực cho `slides.get_frame`.
    - Điều phối ngân sách thông minh (tiết kiệm credit cho các vòng sau nhân hệ số $\times 1.5$) và thu hẹp field mask tối ưu.
  - Cài đặt toàn diện các chốt chặn Guardrails: Chống prompt injection, bóc tách và che giấu dữ liệu riêng tư ($\ge 40$ chars), xác thực số liệu và chính sách từ chối khi không có căn cứ (`Grounding`).
  - Toàn thắng Knockout cả 3 cấp độ Bot trong đấu trường:
    - **Rookie (EASY)**: Thắng Knockout **100 — 0** (Hiệp 6).
    - **Operator (MEDIUM)**: Thắng Knockout **89 — 0** (Hiệp 6).
    - **Adversary (HARD - Trùm cuối)**: Thắng Knockout **48 — 0** (Hiệp 9).
- **Task 3 (PROSECUTE - eval/prosecute.py)**:
  - Xây dựng hoàn chỉnh toàn bộ 16 bộ phát hiện lỗi (detector hooks) thuộc 5 họ lỗi (Infrastructure, Truth, Safety, Quality, Economy).
  - Tích hợp hệ thống quản lý ngân sách luận tội `ProsecutionBudget` (tối đa 4 đơn/hiệp, tối đa 1 đơn/họ lỗi, ngưỡng tin cậy $p \ge 44.4\%$).
  - Đạt điểm tuyệt đối trên toàn bộ 40 fixtures chuẩn: `precision = 1.000` (100%), `recall = 1.000` (100% - 34/34 lỗi), `f1 = 1.000`, `false_claim_rate = 0.000` (0 lỗi vu khống).
  - Vượt qua 41/41 unit tests trong `tests/test_prosecute.py`.




