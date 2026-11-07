# 02 — Deep-Dive Report: Vinhomes Resident Feedback Triage

> **Học viên:** Dương Tuấn Hưng — MSSV: 2A202603416
> **Bài toán:** Phân loại & điều hướng phản ánh cư dân trên App Vinhomes Resident
> **Nội dung:** Phase 3 (DEEP-DIVE) + Phase 5 (EVALUATE)

> [!NOTE]
> Số liệu là **ước tính giả định** cho một khu đô thị quy mô ~30.000 cư dân, dùng để minh hoạ phương pháp. Cần xác lập baseline bằng log thật trước khi build.

---

## 🏗️ 3.1. Current-State Workflow Mapping

Sơ đồ trực quan: xem file [`04-workflow-diagram.png`](04-workflow-diagram.png).

```text
┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Bước 1       │    │ Bước 2       │    │ Bước 3       │    │ Bước 4       │    │ Bước 5       │
│ Cư dân gửi   │    │ CSKH đọc &   │    │ CSKH phân    │    │ Chuyển ticket│    │ Soạn phản hồi│
│ phản ánh     │──→ │ gọi lại hỏi  │──→ │ loại nhóm &  │──→ │ cho tổ xử lý │──→ │ tiếp nhận    │
│ qua App      │ 🔄 │ thêm thông tin│   │ mức khẩn cấp │ 🔄 │ (Zalo/hệ thống)│  │ gửi cư dân   │
│              │    │              │    │              │    │              │    │              │
│ Ai: Cư dân   │    │ Ai: CSKH     │    │ Ai: CSKH     │    │ Ai: CSKH     │    │ Ai: CSKH     │
│ ⏱ 3 phút     │    │ ⏱ 5 phút 🔴  │    │ ⏱ 3 phút 🔴  │    │ ⏱ 2 phút     │    │ ⏱ 3 phút     │
│ In: Text+ảnh │    │ In: Ticket   │    │ In: Ticket đủ│    │ In: Nhãn     │    │ In: Ticket   │
│ Out: Ticket  │    │ Out: Mô tả đủ│    │ Out: Nhãn+P  │    │ Out: Việc giao│   │ Out: Tin nhắn│
└──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘    └──────────────┘
                                         ⚠️ ~20% chuyển sai tổ → quay lại Bước 3 (+30–60 phút)

🔴 Bottleneck   🔄 Handoff (Cư dân → CSKH; CSKH → Tổ kỹ thuật/An ninh/Vệ sinh)
⏱ Tổng thời gian thao tác của CSKH: ~13 phút/ticket (chưa tính thời gian chờ trong hàng đợi)
⏱ Thời gian từ lúc gửi đến khi tổ xử lý nhận việc: trung bình 4–12 giờ (do backlog)
```

**Handoff points:**
1. Cư dân → CSKH (qua App, thường thiếu vị trí cụ thể/tầng/căn).
2. CSKH → Tổ xử lý (Kỹ thuật điện nước, Thang máy, An ninh, Vệ sinh, Cảnh quan, Kế toán phí) — điểm hay bị chuyển sai.

---

## 📋 3.2. Problem Statement (6-field)

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Nhân viên CSKH thuộc Ban Quản lý (BQL) khu đô thị Vinhomes, ca trực 3–4 người/khu. |
| **2. Current Workflow** | Cư dân gửi phản ánh (text tự do + ảnh) trên App Vinhomes Resident → CSKH đọc từng ticket, gọi lại hỏi thêm nếu thiếu thông tin → tự phân loại vào 1 trong 8 nhóm (Điện, Nước, Thang máy, An ninh, Vệ sinh, Cảnh quan, Tiếng ồn/Hàng xóm, Phí dịch vụ) và gán mức ưu tiên → chuyển cho tổ phụ trách qua hệ thống/Zalo → soạn tin xác nhận tiếp nhận. Hoàn toàn thủ công, ~13 phút thao tác/ticket. |
| **3. Bottleneck** | **Bước 2–3 (~8 phút):** đọc hiểu mô tả mơ hồ, đa vấn đề, lẫn cảm xúc; quyết định nhóm & mức khẩn. ~20% ticket bị chuyển sai tổ, phải chuyển lại, kéo dài thời gian xử lý thêm 30–60 phút. Giờ cao điểm (18h–22h) backlog lên tới 40–60 ticket. |
| **4. Business Impact** | ~300 ticket/ngày/khu × 13 phút ≈ **65 giờ công/ngày** chỉ cho tiếp nhận & phân loại. Thời gian phản hồi lần đầu trung bình 4–12 giờ, vượt SLA cam kết 2 giờ. Ticket khẩn cấp (rò điện, kẹt thang máy) có lúc bị chìm trong hàng đợi → rủi ro an toàn & khiếu nại công khai trên mạng xã hội, ảnh hưởng thương hiệu Vinhomes. |
| **5. Success Metric** | 1. **Efficiency:** thời gian phân loại + soạn nháp phản hồi từ 8 phút → **< 1 phút/ticket**.<br>2. **Quality:** tỉ lệ chuyển đúng tổ ngay lần đầu từ ~80% → **≥ 95%** (đo trên tập 500 ticket lịch sử đã gán nhãn).<br>3. **Safety:** **100%** ticket chứa tín hiệu an toàn tính mạng (cháy, khói, rò điện, kẹt thang máy, ngập) được escalate cho con người **< 2 phút**.<br>4. **SLA:** thời gian phản hồi lần đầu < 30 phút cho 90% ticket. |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:** đọc nội dung ticket, đề xuất nhóm + mức ưu tiên + độ tự tin, đề xuất câu hỏi làm rõ, soạn **nháp** tin phản hồi tiếp nhận.<br>**AI TUYỆT ĐỐI KHÔNG ĐƯỢC:** tự gửi tin cho cư dân (mọi output gắn `[DRAFT_ONLY]`); hứa thời gian sửa chữa, bồi thường, miễn/giảm phí; đưa ra kết luận về lỗi/trách nhiệm giữa các hộ; tiết lộ thông tin cá nhân của cư dân khác.<br>**ĐIỂM CẦN NGƯỜI DUYỆT:** mọi tin gửi cư dân; mọi ticket nhóm *Phí dịch vụ* và *Tiếng ồn/Hàng xóm* (nhạy cảm pháp lý); ticket khẩn cấp → bỏ qua AI, báo động trực tiếp cho trưởng ca. |

---

## 🔀 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix

| Phương án | Ưu điểm | Nhược điểm | Kết luận |
|---|---|---|---|
| **Rule / State-Machine** (keyword) | Nhanh, rẻ, giải thích được, deterministic | Vỡ với mô tả mơ hồ, đa vấn đề, tiếng lóng, sai chính tả (~40% ticket) | Dùng cho **lớp an toàn** (từ khóa khẩn cấp) và case rõ ràng |
| **LLM Feature** (classify + draft) | Hiểu ngôn ngữ tự nhiên tiếng Việt, xử lý được case mơ hồ, soạn tin thân thiện | Có thể hallucinate, cần kiểm soát output, có chi phí API | ✅ **Chọn** cho bước phân loại & soạn nháp |
| **Agentic Loop** | Có thể tự điều phối tổ, theo dõi tiến độ | Rủi ro cao, khó kiểm soát, không cần thiết vì quy trình có cấu trúc cố định | ❌ Không chọn (over-engineering) |

**→ Kiến trúc: Hybrid = Rule (safety gate) + LLM Feature + HITL.**

### Future-State Flow

```text
┌──────────────┐   ┌───────────────────┐   ┌──────────────────┐   ┌──────────────────┐   ┌──────────────┐
│ Bước 1       │   │ Bước 2            │   │ Bước 3           │   │ Bước 4           │   │ Bước 5       │
│ Cư dân gửi   │──→│ ⚙️ RULE Safety Gate│──→│ 🔵 AI STEP (LLM) │──→│ 🟢 HUMAN (HITL)  │──→│ Hệ thống     │
│ phản ánh     │   │ Quét từ khóa khẩn │   │ - Phân loại nhóm │   │ CSKH xem đề xuất,│   │ chuyển ticket│
│ qua App      │   │ cấp (cháy, khói,  │   │ - Mức ưu tiên    │   │ sửa nếu cần,     │   │ & gửi tin    │
│              │   │ rò điện, kẹt TM)  │   │ - Độ tự tin      │   │ bấm DUYỆT        │   │ cho cư dân   │
│              │   │                   │   │ - Nháp phản hồi  │   │ ⏱ < 1 phút       │   │              │
│              │   │                   │   │   [DRAFT_ONLY]   │   │                  │   │              │
└──────────────┘   └─────────┬─────────┘   └────────┬─────────┘   └──────────────────┘   └──────────────┘
                             │ khẩn cấp             │ confidence < 0.8 / JSON lỗi /
                             ▼                      │ nhóm Phí dịch vụ, Hàng xóm
                   🚨 Escalate ngay cho             ▼
                   trưởng ca + tổ An ninh   ↩️ FALLBACK: chuyển vào hàng đợi
                   (bỏ qua AI, < 2 phút)       "Cần người phân loại" — CSKH
                                               xử lý thủ công như quy trình cũ
```

- 🔵 **AI Step:** LLM trả JSON `{category, priority, confidence, clarifying_questions, draft_reply}`, draft luôn bắt đầu bằng `[DRAFT_ONLY]`.
- 🟢 **Human Step (HITL):** CSKH duyệt 100% tin gửi cư dân trong giai đoạn pilot; sau 3 tháng nếu precision ≥ 97% có thể cho phép tự chuyển ticket (vẫn duyệt tin nhắn).
- ↩️ **Fallback:** API lỗi/timeout > 5s, JSON không hợp lệ, confidence < 0.8 → quay về quy trình thủ công; không có ticket nào bị "mất" vì AI lỗi.

**Thời gian thao tác dự kiến:** ~13 phút → ~3 phút/ticket (đọc lướt + duyệt + sửa nhỏ).

---

## 💻 Phase 4 — Liên hệ Prompt Prototype

File [`starter-code/prompt_prototype.py`](starter-code/prompt_prototype.py) hiện thực hoá cùng nguyên tắc boundary cho bài toán điều vận Xanh SM do lab cung cấp:
- `[DRAFT_ONLY]` bắt buộc → tương ứng HITL "không tự gửi tin" ở bài toán Vinhomes.
- Ngưỡng an toàn cứng (pin < 5% → `dispatch_mobile_charger`) → tương ứng Rule Safety Gate (tín hiệu khẩn cấp → escalate ngay).
- 3 adversarial tests: ép vượt ngưỡng an toàn, ép bỏ thẻ `[DRAFT_ONLY]`, prompt injection giả danh lãnh đạo.

---

## 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

1. [x] **Có sẵn dữ liệu mẫu/logs để test?** — Có: lịch sử ticket trên App Vinhomes Resident kèm nhãn tổ đã xử lý cuối cùng (cần làm sạch & ẩn danh PII trước khi dùng).
2. [x] **Rủi ro khi AI sai nằm trong tầm kiểm soát?** — Có: Rule Safety Gate xử lý ca khẩn cấp độc lập với AI; 100% tin nhắn qua HITL; fallback về quy trình cũ.
3. [ ] **Stakeholders sẵn sàng thay đổi quy trình?** — Chưa xác nhận: cần pilot với 1 khu và đào tạo CSKH; tổ kỹ thuật cần thống nhất bộ 8 nhóm phân loại chuẩn.

### Quyết định cuối cùng

- [x] **GO (Bắt đầu xây dựng Prototype) — với scope hẹp**
- [ ] NOT YET
- [ ] NO-GO

**Justification:**
> - **Giá trị rõ ràng:** ~65 giờ công/ngày/khu cho tác vụ lặp lại; SLA hiện bị vượt nặng.
> - **Kỹ thuật phù hợp:** bài toán phân loại văn bản tiếng Việt + soạn nháp là điểm mạnh của LLM; rule-based thuần đã được cân nhắc và chỉ phù hợp cho lớp an toàn.
> - **Rủi ro kiểm soát được:** AI không có quyền gửi tin, không đụng tới tiền/phí, ca khẩn cấp đi đường riêng không phụ thuộc AI.
> - **Chi phí thấp:** ước tính ~300 ticket × ~1.000 token ≈ 0,3 triệu token/ngày/khu với Gemini 2.5 Flash — chi phí rất nhỏ so với 65 giờ công.
> - **Điều kiện GO:** chỉ pilot tại **1 khu đô thị trong 4 tuần**, chỉ 6 nhóm ít nhạy cảm (loại trừ *Phí dịch vụ* & *Hàng xóm*), đo baseline 2 tuần trước khi bật AI. Nếu tỉ lệ chuyển đúng tổ < 90% sau pilot → quay về **NOT YET** để bổ sung dữ liệu.

