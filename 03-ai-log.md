# Phase 6 — AI Log & Reflection (Nhật ký tương tác AI)

**Họ và tên / Mã sinh viên:** Tô Minh Thành
**Dự án lựa chọn:** Tối ưu hóa Tóm tắt Hồ sơ Xuất viện (Vinmec)

Trong suốt quá trình làm bài Lab 02 này, em đã sử dụng LLM (Gemini/ChatGPT) như một "Thought-Partner" (người đồng hành tư duy) để lên ý tưởng, phản biện và viết code prototype. Dưới đây là những ghi nhận thực tế của em về hiệu quả cũng như những rủi ro khi làm việc với AI.

### 1. AI đã giúp tôi những gì? (The Good)
* **Brainstorming bài toán thực tế:** Ban đầu em khá bí ý tưởng về mảng Y tế (Vinmec). em đã dùng prompt gợi ý từ tài liệu: *"Tôi là AI Engineer... hãy gợi ý 5 quy trình nghiệp vụ thủ công tốn thời gian"*. AI đã đưa ra ý tưởng về "Discharge Summary" (Tóm tắt xuất viện) cực kỳ hợp lý, chỉ ra đúng nỗi đau của bác sĩ là mất quá nhiều thời gian tổng hợp thông tin rải rác.
* **Cấu trúc hóa ý tưởng:** Khi em viết draft thô cho Problem Statement (6-field), AI đã giúp em gọt giũa lại câu chữ cho chuyên nghiệp hơn, đặc biệt là phần định lượng Business Impact và Success Metric (ví dụ: gợi ý con số giảm từ 30 phút xuống dưới 5 phút).
* **Code Prototype (JSON Output):** AI giúp em viết nhanh bộ khung code Python sử dụng thư viện `google-genai` và ép kiểu đầu ra thành định dạng JSON có cấu trúc (Structured Output), tiết kiệm rất nhiều thời gian so với việc tự viết Regex để parse text.

### 2. AI đã sai ở đâu / Hallucination như thế nào? (The Bad)
* **Vượt ranh giới y khoa (Hallucination):** Trong lần test prototype đầu tiên, em đưa vào một bệnh án mẫu bị thiếu phần hướng dẫn dùng thuốc sau xuất viện. Thay vì báo là "Không có dữ liệu", AI lại tự động "bịa" (hallucinate) ra việc kê đơn Paracetamol và kháng sinh cơ bản dựa trên chẩn đoán sốt. Đây là một lỗi **đỏ (red flag)** cực kỳ nguy hiểm trong y tế.
* **Format không ổn định:** Khi em test bằng Adversarial Prompt (cố tình dụ AI nói chuyện phiếm), thỉnh thoảng AI quên mất việc phải trả về JSON mà lại trả về text thông thường kèm theo JSON, làm crash code Python.

### 3. Tôi đã sửa lỗi và điều chỉnh Prompt ra sao? (The Fix)
Để khắc phục các rủi ro trên, em đã tinh chỉnh System Prompt trong file `prompt_prototype.py` trở nên cực kỳ nghiêm ngặt:

1. **Siết chặt Operational Boundary:** em thêm câu lệnh in hoa: *"BẠN CHỈ LÀ TRỢ LÝ TỔNG HỢP. TUYỆT ĐỐI KHÔNG SÁNG TẠO, KHÔNG KÊ ĐƠN THUỐC, KHÔNG ĐƯA RA CHẨN ĐOÁN MỚI. NẾU THÔNG TIN KHÔNG CÓ TRONG ĐẦU VÀO, HÃY ĐIỀN 'KHÔNG CÓ DỮ LIỆU'."*
2. **Xử lý Adversarial input:** em quy định rõ: *"Nếu người dùng yêu cầu làm những việc ngoài tóm tắt bệnh án (ví dụ: kê thêm thuốc, hỏi thời tiết), hãy trả về JSON với trường status = 'rejected' và reason = 'Vượt quá thẩm quyền'."*
3. **Ép kiểu JSON:** Dùng tính năng `response_schema` của Gemini API để đảm bảo 100% output luôn trả về đúng định dạng, không bị lẫn text rác.

### 4. Bài học rút ra (Takeaway)
Trong các quy trình nghiệp vụ quan trọng (đặc biệt là Y tế như Vinmec), AI (LLM) rất giỏi tóm tắt nhưng lại quá "nhiệt tình" dẫn đến rủi ro bịa đặt thông tin. Do đó, thiết kế một **Operational Boundary** rõ ràng bằng System Prompt và bắt buộc phải có bước **Human-in-the-loop (Bác sĩ phê duyệt trước khi in)** là điều kiện tiên quyết để dự án được phép Go-live.