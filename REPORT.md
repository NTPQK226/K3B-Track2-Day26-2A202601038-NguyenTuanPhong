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

## Trả lời câu hỏi bảo vệ bài (Oral Defense)

### Câu 1: Vì sao Gateway.decide không có execute(), và điều đó bảo vệ BẠN chứ không chỉ bảo vệ trọng tài?
- **Phân định ranh giới tin cậy (Trust Boundary):** `Gateway.decide` chỉ chịu trách nhiệm đưa ra quyết định (`Decision`), còn việc thực thi (`execute`), tính chi phí credit, áp mutation và ghi log L1 hoàn toàn do Arena đáng tin cậy đảm nhiệm.
- **Bảo vệ chính Agent:**
  1. **Chống vu khống:** Đối thủ chỉ có thể kiện bạn dựa trên đúng `Decision` mà Gateway của bạn đã chính thức trả về cho lệnh đó, không thể vu cáo một hành vi bạn không phê duyệt.
  2. **Trace bất biến, không thể bị giả mạo:** Ngăn chặn trường hợp tool hoặc memory độc hại tự ý kích hoạt các lời gọi trái phép ngoài tầm kiểm soát của Gateway.

### Câu 2: act và sub khác nhau ở đâu, và vì sao operator — một bot viết rất hợp lý — vẫn thua chính xác ở chỗ này?
- **Khác biệt:**
  - `sub` (Subject): Định danh cá nhân của Agent ("Bạn là ai" - ví dụ: `agent:vlearn-tutor`).
  - `act` (Actor Context): Quyền lực ủy quyền ("Bạn đang hành động nhân danh người dùng nào" - ví dụ: `learner:sv-0417`).
- **Lý do operator thất bại:**
  - Quyền hạn ghi dữ liệu (Authorization) bắt nguồn từ `act`, không bao giờ từ `sub`.
  - Bot `operator` kiểm tra nhầm danh tính từ `sub` (tin rằng mình là tutor hợp lệ nên được phép ghi) thay vì kiểm tra mục tiêu ghi `target` có thuộc quyền sở hữu của `ctx.act` hay không (`target == ctx.act`). Khi bị tấn công `replace_act` (Confused Deputy), `operator` đã cho phép ghi chéo dữ liệu người dùng khác và bị xử phạt lỗi nặng nhất: `authority_exceeded` (trọng số 10).

### Câu 3: Vì sao trọng số lớp lỗi bị triệt tiêu khỏi ngưỡng break-even 44,4%, và nếu hình phạt là hằng số −4 thì chiến thuật hợp lý sẽ đổi thành gì?
- **Triệt tiêu trọng số:**
  - Kỳ vọng điểm khi tố cáo 1 lỗi có trọng số $w$ với xác suất đúng $p$:
    $$\mathbb{E} = p \cdot w - (1 - p) \cdot (0.8 \cdot w) = w \cdot (1.8p - 0.8)$$
  - Để có lợi nhuận $(\mathbb{E} > 0) \iff 1.8p - 0.8 > 0 \iff p > \frac{0.8}{1.8} = \frac{4}{9} \approx \mathbf{44.4\%}$.
  - Trọng số $w$ bị triệt tiêu ở 2 vế nên ngưỡng hòa vốn $44.4\%$ là đồng nhất cho toàn bộ 17 lớp lỗi.
- **Nếu hình phạt là hằng số $-4$ (flat penalty):**
  - Kỳ vọng: $\mathbb{E} = p \cdot w - (1 - p) \cdot 4 > 0 \iff p > \frac{4}{w + 4}$.
  - Lớp nặng $w=10$ chỉ cần $p > \frac{4}{14} \approx 28.6\%$ là có lãi, trong khi lớp nhẹ $w=3$ cần $p > \frac{4}{7} \approx 57.1\%$.
  - **Chiến thuật biến chất thành:** *"Bắn bừa (shotgunning) vào các lớp điểm cao (10 và 8) kể cả khi độ tự tin rất thấp vì tiền phạt cố định quá rẻ so với phần thưởng lớn"*. Cơ chế phạt tỉ lệ $-0.8w$ của giải đấu đã triệt tiêu hoàn toàn lỗ hổng này.





