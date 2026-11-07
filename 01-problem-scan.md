# 01 — Problem Scan & Quick Assess (Cá nhân)

> **Vai trò:** AI Product Engineer tại **Vin Smart Future**
> **Phạm vi:** Phase 1 (SCAN) + Phase 2 (QUICK-ASSESS) theo `01-worksheet.md`

---

## 🔍 Phase 1 — SCAN: Bảng quét cơ hội

Sử dụng 4 lenses: **Lặp lại · Tốn thời gian · AI-upgrade · Stakeholder Pain** (đã dùng đủ cả 4 lenses).

| # | Subsidiary | Lens | Mô tả ngắn bài toán thực tế |
|---|------------|------|-----------------------------|
| 1 | **Xanh SM (GSM)** | Tốn thời gian | Điều phối viên xử lý thủ công các báo cáo sự cố hết pin / lỗi trạm sạc từ tài xế thực địa, tra cứu tọa độ GPS và trụ sạc trống (mất **12–15 phút/lượt**). |
| 2 | **Vinhomes** | Lặp lại | Phân loại và điều hướng thủ công hàng nghìn phản ánh, khiếu nại của cư dân từ App Vinhomes Resident đến đúng Ban quản lý từng tòa nhà. |
| 3 | **VinFast** | Lặp lại | So khớp và đối chiếu thủ công hóa đơn sạc điện hằng tuần từ các trụ sạc đối tác liên kết ngoài với hệ thống kế toán tài chính. |
| 4 | **Vinmec** | Stakeholder Pain | Bác sĩ mất **20–30 phút/bệnh nhân** viết tóm tắt hồ sơ xuất viện (Discharge Summary) từ dữ liệu bệnh án điện tử, gây quá tải y tế. |
| 5 | **Xanh SM (GSM)** | AI-upgrade | Phân tích tự động lý do khách hàng hủy chuyến từ ghi âm cuộc gọi và ghi chú tài xế để phát hiện rò rỉ cuốc xe theo thời gian thực. |

### Sàng lọc nhanh (để chọn top 3)

Chấm điểm 1–5 (5 = tốt nhất cho một prototype AI trong ngắn hạn).

| # | Mức độ đau (Pain) | Tần suất | Dữ liệu sẵn có | Rủi ro khi AI sai (5 = thấp) | AI có lợi thế hơn Rule? | **Tổng** |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| 1 — Xanh SM sự cố pin | 5 | 5 | 4 | 4 (có HITL) | 4 | **22** |
| 2 — Vinhomes route khiếu nại | 4 | 5 | 4 | 3 | 4 | **20** |
| 3 — VinFast đối chiếu hóa đơn | 3 | 3 | 4 | 3 | 2 (Rule/RPA đủ) | **15** |
| 4 — Vinmec Discharge Summary | 5 | 4 | 3 (dữ liệu nhạy cảm) | 2 (rủi ro lâm sàng) | 5 | **19** |
| 5 — Xanh SM phân tích hủy chuyến | 3 | 4 | 2 (cần ASR, chưa gán nhãn) | 4 | 4 | **17** |

👉 **Top 3 được chọn:** **#1 (Xanh SM sự cố pin)**, **#2 (Vinhomes route khiếu nại)**, **#4 (Vinmec Discharge Summary)**.

* **Loại #3 (VinFast hóa đơn):** dữ liệu có cấu trúc (mã trụ, kWh, thời gian, đơn giá) → so khớp bằng **Rule/RPA + SQL** là đủ và kiểm toán được. Dùng LLM ở đây tăng chi phí và rủi ro sai số tài chính mà không thêm giá trị.
* **Loại #5 (Hủy chuyến):** giá trị cao nhưng phụ thuộc pipeline Speech-to-Text tiếng Việt và dữ liệu chưa gán nhãn lý do hủy → nên để giai đoạn sau, sau khi có baseline.

---

## 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### Card #1 — Xanh SM: Xử lý sự cố hết pin / lỗi trạm sạc thực địa

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán (1 câu): Điều phối viên mất 12-15 phút/lượt để xử  │
│ lý thủ công báo cáo hết pin/lỗi trụ sạc của tài xế (tra GPS,│
│ tìm trụ trống đúng chuẩn sạc, soạn chỉ dẫn).                 │
│ Công ty thành viên: [x] Xanh SM (GSM)                       │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│  - Điều phối viên (Dispatcher) – quá tải giờ cao điểm       │
│  - Tài xế – xe đứng chờ, mất cuốc, mất thu nhập              │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                         │
│  1. Tài xế gọi hotline / chat báo sự cố                      │
│  ──> 2. Dispatcher tra biển số → vị trí GPS + % pin xe       │
│  ──> 3. Mở Dashboard trạm sạc, lọc thủ công trụ còn trống,   │
│         đúng chuẩn sạc, trong bán kính an toàn               │
│  ──> 4. Soạn tin chỉ dẫn (địa chỉ, khoảng cách, trụ số mấy)  │
│  ──> 5. Nếu pin quá thấp → gọi cứu hộ / xe sạc di động       │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3 + 4                  │
│   (⏱ ~9-10 phút/lượt; lỗi hay gặp: trụ hiển thị "trống" nhưng│
│    thực tế đang hỏng, gợi ý trạm quá xa so với % pin)        │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                        │
│   Bước 3: Rule/API lọc trạm (khoảng cách, chuẩn sạc, trạng   │
│   thái). Bước 4: LLM soạn draft chỉ dẫn + gợi ý action       │
│   (station / mobile charger) → Dispatcher bấm duyệt.        │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - Giảm thời gian xử lý từ 12-15 phút ──> dưới 3 phút/lượt  │
│   - ≥ 98% đề xuất đúng trạm/đúng chuẩn sạc (audit mẫu)       │
│   - 100% ca pin < 5% được chuyển sang xe sạc di động         │
│                                                             │
│ Quick Architecture: [ ] No AI  [x] Rule  [x] LLM  [ ] Agent │
│   (Rule lọc dữ liệu cứng + LLM soạn nháp, có HITL)           │
└─────────────────────────────────────────────────────────────┘
```

### Card #2 — Vinhomes: Phân loại & điều hướng phản ánh cư dân

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán (1 câu): CSKH phải đọc và phân loại thủ công hàng  │
│ nghìn phản ánh/ngày trên App Vinhomes Resident rồi chuyển    │
│ đến đúng Ban quản lý (BQL) tòa nhà và đúng bộ phận.          │
│ Công ty thành viên: [x] Vinhomes                            │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│  - Nhân viên CSKH tổng (phân loại thủ công)                 │
│  - Cư dân (chờ phản hồi lâu), BQL (nhận sai ticket)         │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                         │
│  1. Cư dân gửi phản ánh (text + ảnh) trên App               │
│  ──> 2. CSKH đọc, xác định loại (điện nước, vệ sinh, an ninh,│
│         phí dịch vụ, tiếng ồn...) và mức độ khẩn             │
│  ──> 3. Tra mã căn hộ → tòa nhà → BQL phụ trách, gán ticket  │
│  ──> 4. BQL xử lý; nếu sai tuyến thì trả về, gán lại         │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 (⏱ ~3-5 phút/ticket, │
│   và ticket xếp hàng chờ nhiều giờ vào cuối tuần); sai tuyến │
│   ở bước 4 làm kéo dài thêm 1 vòng xử lý.                    │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                        │
│   Bước 2: LLM phân loại category + urgency + tóm tắt 1 dòng. │
│   Bước 3: Rule tra mã căn hộ → BQL (không cần AI).          │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - ≥ 90% ticket được phân loại đúng category trong < 10 giây│
│   - Giảm tỉ lệ route sai từ ~15% (ước tính) ──> < 5%         │
│   - 100% ticket khẩn (an ninh, cháy nổ, rò điện) đến người   │
│     trực trong < 5 phút                                      │
│                                                             │
│ Quick Architecture: [ ] No AI  [x] Rule  [x] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
```

### Card #3 — Vinmec: Soạn nháp tóm tắt hồ sơ xuất viện (Discharge Summary)

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán (1 câu): Bác sĩ mất 20-30 phút/bệnh nhân tổng hợp   │
│ dữ liệu EMR để viết Discharge Summary, gây quá tải và chậm   │
│ thủ tục xuất viện.                                           │
│ Công ty thành viên: [x] Vinmec                              │
│                                                             │
│ Ai đang đau (Actor)?                                        │
│  - Bác sĩ điều trị (viết tóm tắt ngoài giờ)                 │
│  - Bệnh nhân / người nhà (chờ giấy ra viện)                 │
│                                                             │
│ Workflow thủ công hiện tại (4 bước):                         │
│  1. Bác sĩ mở EMR: diễn biến, xét nghiệm, CĐHA, y lệnh       │
│  ──> 2. Đọc và chọn lọc thông tin chính trong N ngày nằm viện│
│  ──> 3. Viết tóm tắt: chẩn đoán, điều trị, thuốc ra viện,    │
│         dặn dò tái khám                                      │
│  ──> 4. Trưởng khoa/bác sĩ ký duyệt, in cho bệnh nhân        │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 + 3 (⏱ 20-25 phút)   │
│ AI có thể nhảy vào hỗ trợ ở bước nào?                        │
│   Bước 2-3: LLM soạn nháp có trích dẫn nguồn từng câu về     │
│   bản ghi EMR; bác sĩ sửa & ký (bắt buộc HITL).              │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   - Giảm thời gian viết từ 20-30 phút ──> dưới 7 phút/BN      │
│   - 0 lỗi sai thuốc/liều trong nháp (audit 100% giai đoạn    │
│     pilot)                                                   │
│   - ≥ 80% nháp được bác sĩ chấp nhận với sửa đổi nhỏ          │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
│   (LLM + RAG trên EMR, bắt buộc bác sĩ ký duyệt)             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🧪 Stress-test các card (phản biện vai CFO / Trưởng phòng Vận hành)

| Card | Điểm yếu bị chỉ ra | Cách tôi điều chỉnh |
|---|---|---|
| #1 Xanh SM | "Tìm trạm sạc gần nhất là bài toán khoảng cách + filter – Rule làm được, cần gì LLM?" | Đồng ý một phần: **bước lọc trạm giao cho Rule/API** (deterministic). LLM chỉ đảm nhận đọc báo cáo tự do của tài xế (text/voice-to-text lộn xộn) + soạn chỉ dẫn + giải thích lý do. Không để LLM tự tính khoảng cách. |
| #1 Xanh SM | Số "80 sự cố/ngày" chưa có nguồn. | Ghi rõ là **giả định**, cần xác lập baseline bằng log tổng đài 2 tuần trước khi cam kết ROI. |
| #2 Vinhomes | Phân loại category có thể làm bằng keyword rule. | Rule bắt được ~60% ca rõ ràng; phản ánh của cư dân thường đa ý, dùng từ địa phương, kèm cảm xúc → LLM xử lý tốt phần còn lại. Đề xuất **Rule trước, LLM cho ca mơ hồ**. |
| #3 Vinmec | Rủi ro lâm sàng & pháp lý quá cao, dữ liệu bệnh nhân nhạy cảm. | Giữ nguyên ở mức **NOT YET** cho đến khi có phê duyệt của Hội đồng đạo đức, hạ tầng LLM on-premise và bộ dữ liệu đánh giá do bác sĩ gán nhãn. |

---

## 🗳️ Đề xuất chọn bài toán Deep-Dive

**Chọn Card #1 — Xanh SM: Xử lý sự cố hết pin / lỗi trạm sạc thực địa.**

* Tần suất cao, ảnh hưởng **trực tiếp, thời gian thực** đến doanh thu (xe đứng = mất cuốc).
* Dữ liệu đầu vào phần lớn đã có qua API (GPS, % pin, trạng thái trụ sạc) → prototype nhanh.
* Rủi ro kiểm soát được bằng **HITL** (Dispatcher duyệt) + **Rule cứng** (pin < 5% → xe sạc di động).
* Card #2 là ứng viên thứ hai tốt (rủi ro thấp, ROI rõ) – đề xuất làm sprint sau. Card #3 giá trị lớn nhất nhưng chưa sẵn sàng về pháp lý và dữ liệu.
