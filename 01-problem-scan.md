# 01 — Problem Scan & Quick Cards (Vin Smart Future)

## Phase 1 — SCAN: 5 bài toán

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | Vin Smart Future (nhân sự) | Tốn thời gian + Lặp lại | Chuyên viên tuyển dụng đọc hàng trăm CV cho mỗi vị trí kỹ sư, đối chiếu thủ công với JD để chọn người mời phỏng vấn. |
| 2 | Xanh SM | Lặp lại | Điều phối viên soạn tin nhắn trấn an/thông báo cho tài xế và khách khi chuyến bị trễ hoặc đổi xe, lặp lại hàng trăm lần/ngày. |
| 3 | Vinhomes | Tốn thời gian | Nhân viên CSKH đọc và soạn phản hồi cho phản ánh của cư dân (thang máy, gửi xe, tiếng ồn), mỗi ca mất nhiều phút. |
| 4 | Vinmec | Tốn thời gian | Điều dưỡng/lễ tân tóm tắt hồ sơ bệnh án và nhắc lịch tái khám bằng cách đọc, nhập tay. |
| 5 | Vinpearl / VinWonders | AI có thể tốt hơn | Chatbot CSKH đặt vé/phòng trả lời rập khuôn, khách phải chuyển sang nhân viên khi hỏi ngoài kịch bản. |

## Phase 2 — 3 Quick Problem Cards

### CARD #1 — Sàng lọc CV tuyển kỹ sư (Vin Smart Future)
- **Bài toán:** Chuyên viên tuyển dụng mất nhiều giờ đọc từng CV để chọn ứng viên phù hợp với JD, trong khi vị trí kỹ sư cần lấp nhanh để thu hút và giữ chân nhân tài.
- **Công ty:** [x] Khác: Vin Smart Future (khối nhân sự / Talent Acquisition)
- **Actor:** Chuyên viên tuyển dụng (HR), hiring manager.
- **Workflow hiện tại:** 1. Nhận CV từ nhiều kênh (email, web tuyển dụng) ──> 2. Nhập vào bảng theo dõi ──> 3. Đọc CV, đối chiếu JD ──> 4. Gửi shortlist cho hiring manager ──> 5. Manager chọn người phỏng vấn.
- **Bước tốn thời gian/lỗi nhất:** Bước 3, đọc và đối chiếu (⏱ ~6 phút/CV, *ước tính*); ứng viên tốt có thể bị bỏ sót khi đọc vội.
- **AI nhảy vào ở:** Trích xuất thông tin CV thành bảng có cấu trúc và tóm tắt mức khớp JD kèm trích dẫn; HR duyệt trước khi mời phỏng vấn.
- **Metric:** Giảm thời gian sàng lọc từ ~10 phút/CV xuống dưới 3 phút; ≥ 90% ứng viên HR chọn nằm trong top 20% do AI gợi ý; 0 trường hợp AI tự loại ứng viên.
- **Quick Architecture:** [ ] No AI  [x] Rule  [x] LLM  [ ] Agent  (rule lọc điều kiện cứng và chấm điểm theo rubric, LLM chỉ đọc và tóm tắt)

### CARD #2 — Soạn phản hồi phản ánh cư dân (Vinhomes)
- **Bài toán:** CSKH phải đọc và soạn phản hồi thủ công cho các phản ánh/đánh giá tiêu cực của cư dân.
- **Công ty:** [x] Vinhomes
- **Actor:** Nhân viên CSKH, ban quản lý tòa nhà.
- **Workflow hiện tại:** 1. Nhận phản ánh (app/hotline) ──> 2. Đọc và phân loại ──> 3. Soạn phản hồi ──> 4. Trưởng nhóm duyệt và gửi.
- **Bước tốn thời gian/lỗi nhất:** Bước 3 soạn phản hồi (⏱ ~10 phút/ca, *ước tính*).
- **AI nhảy vào ở:** Phân loại chủ đề + soạn nháp phản hồi theo giọng thương hiệu.
- **Metric:** Giảm thời gian soạn từ 10 phút xuống dưới 2 phút; ≥ 85% nháp được duyệt mà chỉ sửa nhẹ.
- **Quick Architecture:** [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent

### CARD #3 — Tóm tắt hồ sơ và nhắc tái khám (Vinmec)
- **Bài toán:** Nhân viên y tế mất thời gian đọc hồ sơ dài để tóm tắt và soạn nhắc lịch tái khám.
- **Công ty:** [x] Vinmec
- **Actor:** Điều dưỡng, lễ tân, bác sĩ.
- **Workflow hiện tại:** 1. Mở hồ sơ ──> 2. Đọc lịch sử khám ──> 3. Ghi tóm tắt ──> 4. Gọi/nhắn nhắc lịch.
- **Bước tốn thời gian/lỗi nhất:** Bước 2–3 đọc và tóm tắt (⏱ ~8 phút/hồ sơ, *ước tính*).
- **AI nhảy vào ở:** Tóm tắt nháp để bác sĩ/điều dưỡng kiểm tra; không tự đưa ra chẩn đoán.
- **Metric:** Giảm thời gian chuẩn bị từ 8 phút xuống dưới 3 phút; 0 trường hợp AI tự gửi thông tin y tế chưa duyệt.
- **Quick Architecture:** [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent  (rủi ro cao ⇒ bắt buộc HITL)
