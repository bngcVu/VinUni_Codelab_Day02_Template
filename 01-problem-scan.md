# 01 — Problem Scan & Quick Problem Cards

> **Học viên:** Dương Tuấn Hưng — MSSV: 2A202603416
> **Vai trò:** AI Product Engineer @ Vin Smart Future (Vingroup)
> **Nội dung:** Phase 1 (SCAN) + Phase 2 (QUICK-ASSESS) trong `01-worksheet.md`

> [!NOTE]
> Các con số trong file là **ước tính giả định** dựa trên quan sát công khai và kinh nghiệm sử dụng dịch vụ, dùng để minh hoạ cách scoping. Cần xác minh lại bằng log/dữ liệu thật trước khi triển khai.

---

## 🔍 Phase 1 — SCAN: Danh sách bài toán

Sử dụng 4 Lenses: **Lặp lại · Tốn thời gian · AI có thể tốt hơn · Pain từ người khác**.

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **Vinhomes** | Lặp lại | Ban Quản lý (BQL) phân loại thủ công hàng trăm phản ánh/ngày trên App Vinhomes Resident (mất nước, thang máy, ồn, vệ sinh...) rồi chuyển cho đúng tổ kỹ thuật/an ninh. Nhiều ticket bị chuyển sai, cư dân phải chờ 12–24h. |
| 2 | **VinFast** | AI có thể tốt hơn | Khách hàng mô tả lỗi xe bằng tiếng Việt tự nhiên (*"đi qua gờ kêu cụp cụp bánh trước"*), tổng đài viên phải hỏi lại nhiều lần mới tạo được phiếu dịch vụ có mã lỗi sơ bộ cho xưởng. |
| 3 | **Vinmec** | Pain từ người khác | Bác sĩ mất 20–30 phút/bệnh nhân để viết tóm tắt xuất viện; bệnh nhân lại khó hiểu thuật ngữ y khoa trong bản tóm tắt. |
| 4 | **Vinpearl** | Tốn thời gian | Nhân viên Sales đọc email đặt phòng theo đoàn (group booking) dài, nhiều điều kiện, rồi nhập tay vào hệ thống để kiểm tra quỹ phòng — mất 30–45 phút/email. |
| 5 | **Xanh SM** | Pain từ người khác | Tài xế phàn nàn điểm đón gợi ý sai (cổng sau chung cư, trong ngõ cấm), dẫn tới gọi điện qua lại với khách và khách hủy chuyến. |
| 6 | **VinFast** | Lặp lại | Kế toán đối chiếu hóa đơn sạc điện hằng tuần giữa dữ liệu trụ sạc và hóa đơn đối tác (Excel vs. PDF), dễ sai sót khi nhập tay. |

---

## 🃏 Phase 2 — QUICK-ASSESS: Top 3 Quick Problem Cards

Chọn top 3: **#1 (Vinhomes phản ánh cư dân)**, **#2 (VinFast chẩn đoán lỗi từ mô tả)**, **#4 (Vinpearl group booking)**.

### Card #1 — Vinhomes: Phân loại & điều hướng phản ánh cư dân

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán: Phản ánh của cư dân trên App Vinhomes Resident    │
│ bị phân loại & chuyển tổ xử lý thủ công, chậm và hay sai.   │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên CSKH của BQL tòa nhà         │
│ (quá tải) + Cư dân (chờ lâu) + Tổ kỹ thuật (nhận sai việc). │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Cư dân gửi phản ánh (text + ảnh) qua App               │
│   ──> 2. CSKH đọc, gọi lại hỏi thêm nếu thiếu thông tin     │
│   ──> 3. CSKH tự phân loại (Điện/Nước/Thang máy/An ninh...) │
│        và mức độ khẩn cấp                                   │
│   ──> 4. Chuyển ticket cho tổ phụ trách qua Zalo/Hệ thống   │
│   ──> 5. Soạn tin phản hồi tiếp nhận cho cư dân             │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3                   │
│ (⏱ ~8 phút/ticket, ~20% ticket bị chuyển sai tổ)            │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 + Bước 5       │
│ (phân loại + chấm mức khẩn + soạn nháp phản hồi)            │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian phân loại: 8 phút ──> dưới 1 phút/ticket      │
│   - Tỉ lệ chuyển đúng tổ ngay lần đầu: 80% ──> ≥ 95%        │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

### Card #2 — VinFast: Chẩn đoán lỗi sơ bộ từ mô tả tiếng Việt

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán: Tổng đài viên mất nhiều thời gian chuyển mô tả    │
│ lỗi bằng tiếng Việt tự nhiên thành phiếu dịch vụ có mã lỗi. │
│ Công ty thành viên: [x] VinFast                             │
│                                                             │
│ Ai đang đau (Actor)? Tổng đài viên CSKH VinFast, Cố vấn     │
│ dịch vụ tại xưởng (nhận phiếu mơ hồ).                       │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Khách gọi hotline mô tả hiện tượng                     │
│   ──> 2. Tổng đài viên hỏi lại nhiều câu để làm rõ          │
│   ──> 3. Tra tài liệu, chọn nhóm lỗi (gầm/điện/pin/phần mềm)│
│   ──> 4. Tạo phiếu, đặt lịch xưởng                          │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ ~10 phút/lượt) │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2-3: gợi ý câu   │
│ hỏi làm rõ + đề xuất nhóm lỗi sơ bộ (KHÔNG kết luận lỗi).   │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian tạo phiếu: 10 phút ──> dưới 4 phút            │
│   - Tỉ lệ nhóm lỗi sơ bộ khớp kết luận của xưởng ≥ 85%      │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

### Card #3 — Vinpearl: Đọc email Group Booking

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán: Sales đọc & nhập tay email đặt phòng theo đoàn    │
│ dài, nhiều điều kiện để kiểm tra quỹ phòng.                 │
│ Công ty thành viên: [x] Khác: Vinpearl                      │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Sales/Reservation Vinpearl.  │
│                                                             │
│ Workflow thủ công hiện tại:                                 │
│   1. Nhận email từ công ty lữ hành                          │
│   ──> 2. Đọc & ghi chú số phòng, loại phòng, ngày, điều kiện│
│   ──> 3. Nhập vào PMS kiểm tra phòng trống                  │
│   ──> 4. Soạn email báo giá/xác nhận                        │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2-3 (⏱ ~35 phút/email)│
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2: trích xuất    │
│ thông tin thành JSON có cấu trúc; Bước 4: draft email.      │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       │
│   - Thời gian xử lý: 35 phút ──> dưới 10 phút/email         │
│   - Độ chính xác trích xuất trường (ngày, số phòng) ≥ 98%   │
│                                                             │
│ Quick Architecture: [ ] No AI [ ] Rule [x] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧪 Stress-test bằng AI (CFO khắt khe)

Sau khi dán Card #1 vào LLM với prompt "đóng vai CFO & Trưởng phòng Vận hành khắt khe", các phản biện chính nhận được và cách mình điều chỉnh:

| Phản biện | Điều chỉnh |
|---|---|
| "Phân loại 8–10 nhóm cố định thì rule keyword là đủ, cần gì LLM?" | Rule keyword xử lý được ~60% case rõ ràng (*"mất nước"*), nhưng thất bại với mô tả mơ hồ/đa vấn đề (*"nhà tắm tầng trên rỉ xuống trần nhà em, gọi bảo vệ không ai lên"*) → **Hybrid:** rule cho case rõ, LLM cho case còn lại. |
| "Metric 95% đo trên dữ liệu nào?" | Bổ sung: đo trên tập 500 ticket lịch sử đã được BQL gán nhãn đúng. |
| "Sai mức khẩn cấp (cháy, kẹt thang máy) thì sao?" | Bổ sung boundary: từ khóa an toàn tính mạng → **luôn escalate ngay cho người**, không chờ AI. |

## ✅ Kết luận chọn bài toán cho Deep-Dive

Chọn **Card #1 — Vinhomes** vì: tần suất cao (hàng trăm ticket/ngày), dữ liệu lịch sử sẵn có trên App, rủi ro kiểm soát được bằng HITL, và metric đo được rõ ràng.

- **Loại Card #2:** Rủi ro an toàn xe cao — chẩn đoán sai có thể khiến khách tiếp tục lái xe lỗi; cần dữ liệu kỹ thuật từ xưởng mà nhóm chưa tiếp cận được.
- **Loại Card #3:** Tần suất thấp hơn (vài chục email/tuần), giá trị lớn nhưng phụ thuộc tích hợp PMS phức tạp.

