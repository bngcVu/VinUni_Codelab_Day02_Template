# 02 — Deep-Dive Report: AI hỗ trợ sàng lọc CV tuyển kỹ sư (Vin Smart Future)

> ⚠️ BẢN NHÁP: mọi con số (thời gian, số CV, tỷ lệ) là **ước tính giả định** để minh họa, chưa đo từ dữ liệu thật. Cần kiểm chứng với bộ phận Talent Acquisition trước khi dùng làm căn cứ.

## 1. Current-State Workflow (G1)

Sơ đồ chi tiết: xem `04-workflow-diagram.png`.

| Bước | Ai làm | Việc | Thời gian (ước tính) | Ghi chú |
|---|---|---|---|---|
| 1 | Hệ thống / HR | Nhận CV từ email và web tuyển dụng | ~2 phút/CV (tải, đặt tên) | 🔄 Handoff: từ ứng viên sang HR |
| 2 | HR | Nhập thông tin vào bảng theo dõi | ~2 phút/CV | Nhập tay, dễ sai |
| 3 | HR | Đọc CV, đối chiếu JD, ghi chú | ~6 phút/CV | 🔴 **Bottleneck**: tốn thời gian và dễ bỏ sót |
| 4 | HR → Hiring manager | Gửi shortlist | chờ ~1 ngày | 🔄 Handoff: giữa hai bộ phận |
| 5 | Hiring manager | Chọn người mời phỏng vấn | ~1 ngày | Phản hồi thường chậm |

**Tổng thời gian HR xử lý ≈ 10 phút/CV.** Với ~200 CV cho mỗi vị trí ⇒ ~33 giờ HR/vị trí, chưa tính 1–2 ngày chờ giữa các bộ phận.

## 2. Problem Statement 6 trường (G2)

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Chuyên viên tuyển dụng (HR) của Vin Smart Future; người nhận kết quả là hiring manager. |
| **2. Current Workflow** | Nhận CV qua email/web ──> nhập bảng tính ──> đọc và đối chiếu JD ──> gửi shortlist ──> manager chọn. Công cụ: email, Excel/Google Sheets. |
| **3. Bottleneck** | Bước đọc và đối chiếu CV với JD (~6 phút/CV). Việc lặp lại, đọc vội thì bỏ sót ứng viên tốt. |
| **4. Business Impact** | ~33 giờ HR/vị trí kỹ sư (200 CV × 10 phút). Vị trí để trống lâu hơn, nhân tài có thể nhận offer nơi khác trước khi được liên hệ. |
| **5. Success Metric** | (a) Thời gian sàng lọc từ ~10 phút xuống **dưới 3 phút/CV**. (b) **≥ 90%** ứng viên HR chọn mời phỏng vấn nằm trong top 20% AI gợi ý. (c) **100%** quyết định loại ứng viên do con người đưa ra. |
| **6. Operational Boundary** | **AI được:** trích xuất thông tin từ CV, tóm tắt mức khớp JD kèm trích dẫn, xếp hạng gợi ý. **AI tuyệt đối không:** tự loại hoặc từ chối ứng viên; tự gửi email cho ứng viên; dùng tuổi, giới tính, ảnh, quê quán, tình trạng hôn nhân để đánh giá; bịa thông tin không có trong CV. **Cần duyệt:** mọi danh sách mời phỏng vấn do HR xác nhận. |

## 3. AI-Fit Matrix (G3)

| Tiêu chí | Rule / State-machine | LLM Feature | Agentic Loop |
|---|---|---|---|
| Lọc điều kiện cứng (bằng cấp, số năm kinh nghiệm, chứng chỉ) | ✅ Phù hợp nhất, rẻ, giải thích được | Không cần | Không cần |
| Đọc CV tự do (mỗi người một format, tiếng Việt/Anh) | ❌ Rất khó viết rule | ✅ Phù hợp | Thừa |
| Tóm tắt mức khớp JD | ❌ | ✅ Phù hợp | Thừa |
| Chấm điểm cuối cùng | ✅ Rubric công khai, kiểm toán được | Rủi ro thiên vị, khó giải thích | Rủi ro cao |
| Tự liên hệ ứng viên, đặt lịch | — | — | Không nên, rủi ro cao, cần người |

**Kết luận: Hybrid Rule + LLM Feature.** Rule lọc và chấm điểm (minh bạch), LLM chỉ đọc và tóm tắt (việc rule làm kém). **Không dùng Agent** vì không có bước nào cần tự quyết nhiều vòng, và rủi ro khi sai là loại oan người.

## 4. Future-State Flow (G3)

```
Ứng viên nộp CV
      │
      ▼
[Rule] Kiểm tra định dạng, loại CV trùng                    (tự động)
      │
      ▼
[Rule] Lọc điều kiện cứng (đánh dấu, KHÔNG xóa)             (tự động)
      │
      ▼
🔵 [AI/LLM] Trích xuất → JSON (kỹ năng, kinh nghiệm, học vấn)
            + tóm tắt mức khớp JD, mỗi ý kèm trích dẫn từ CV
      │
      ├── lỗi JSON / CV scan không đọc được / độ tin cậy thấp
      │        └──> ↩️ FALLBACK: chuyển HR đọc thủ công
      ▼
[Rule] Chấm điểm theo rubric công khai ──> xếp hạng gợi ý
      │
      ▼
🟢 [HUMAN / HITL] HR xem danh sách + trích dẫn,
                  xác nhận hoặc đổi thứ hạng, quyết định mời phỏng vấn
      │
      ▼
Hiring manager nhận shortlist đã duyệt
```

- 🔵 **AI Step:** trích xuất và tóm tắt.
- 🟢 **Human Step:** HR duyệt toàn bộ danh sách; bắt buộc trước khi liên hệ ứng viên.
- ↩️ **Fallback:** mọi trường hợp AI lỗi hoặc không chắc đều về HR đọc tay, quy trình cũ vẫn chạy được.
- **Kiểm soát thiên vị:** ẩn tên, ảnh, tuổi, giới tính trước khi đưa vào LLM; định kỳ kiểm tra tỷ lệ chọn theo nhóm.

## 5. Evaluate (G4)

### AI Readiness Checklist
1. [ ] **Dữ liệu mẫu/logs sạch để test?** Chưa chắc: cần xin CV cũ đã ẩn danh kèm kết quả HR từng chọn để làm bộ kiểm thử.
2. [x] **Rủi ro khi AI sai nằm trong tầm kiểm soát?** Có: AI không tự loại ai, HR duyệt hết, có fallback thủ công.
3. [ ] **Stakeholders sẵn sàng đổi quy trình?** Chưa rõ: cần hỏi HR có chấp nhận làm việc với trích dẫn của AI không, và pháp chế về dữ liệu cá nhân ứng viên.

### Quyết định: [x] **GO (phạm vi hẹp)**   [ ] NOT YET   [ ] NO-GO

**Justification:**
- Bước đọc CV đúng là việc LLM làm tốt hơn rule, và rủi ro được chặn bởi thiết kế (AI chỉ gợi ý, người quyết định).
- **Điều kiện của GO:** chạy thử ở chế độ song song (shadow mode) trên **1 vị trí kỹ sư**: HR vẫn làm như cũ, AI chạy ngầm, so sánh kết quả.
- **Tuần đầu bắt buộc đo baseline thật** (thời gian/CV, số CV/vị trí). Nếu số thật khác nhiều so với giả định ở trên thì xem lại metric.
- Nếu kiểm tra thiên vị phát hiện lệch giữa các nhóm ứng viên, **dừng và chuyển sang NO-GO** cho tới khi sửa được.
- **Cần xác nhận với pháp chế** trước khi đưa CV thật vào dịch vụ LLM bên ngoài (dữ liệu cá nhân).
