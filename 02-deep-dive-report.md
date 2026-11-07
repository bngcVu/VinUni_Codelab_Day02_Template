# 02 — Deep-Dive Report: Xanh SM — Trợ lý xử lý sự cố pin & trạm sạc thực địa

> **Vai trò:** AI Product Engineer tại **Vin Smart Future**
> **Phạm vi:** Phase 3 (DEEP-DIVE) + Phase 5 (EVALUATE) theo `01-worksheet.md`
> **Bài toán được chọn:** Card #1 trong [01-problem-scan.md](01-problem-scan.md)
> **Sơ đồ quy trình hiện tại:** [04-workflow-diagram.png](04-workflow-diagram.png)

> ⚠️ **Ghi chú về số liệu:** Các con số về tần suất sự cố, thời gian từng bước và tổn thất là **ước tính dựa trên mô tả bài toán (12–15 phút/lượt) và giả định hợp lý**, chưa phải dữ liệu đo thực tế từ Xanh SM. Bước đầu tiên của dự án là đo baseline thật (xem mục 5).

---

## 🏗️ 3.1. Current-State Workflow Mapping

**Bối cảnh:** Trung tâm Điều vận Xanh SM (ví dụ khu vực Hà Nội). Tài xế gặp sự cố pin yếu / hết pin, hoặc đến trạm sạc thì trụ hỏng / hết chỗ, phải liên hệ tổng đài để được hướng dẫn.

```mermaid
sequenceDiagram
    autonumber
    actor TX as 🚕 Tài xế
    actor DP as 🎧 Dispatcher
    participant FM as 🗺️ Fleet Map
    participant DB as 🔌 Dashboard trạm sạc
    actor CH as 🚛 Đội cứu hộ

    TX->>DP: 🔄 Gọi hotline / chat báo hết pin, trụ lỗi
    Note over DP: Hỏi biển số, mô tả<br/>ghi log sự cố ⏱ 2 phút

    DP->>FM: 🔄 Tra biển số
    FM-->>DP: Tọa độ GPS, % pin, dòng xe
    Note over DP,FM: ⏱ 2 phút — chuyển qua lại nhiều màn hình

    rect rgba(255, 99, 71, 0.18)
        Note over DP,DB: 🔴 BOTTLENECK #1 ⏱ 4–5 phút
        DP->>DB: 🔄 Lọc thủ công trụ trống, đúng chuẩn sạc, đủ gần
        DB-->>DP: Danh sách trạm (không có filter theo % pin / tầm đi)
    end

    rect rgba(255, 99, 71, 0.18)
        Note over TX,DP: 🔴 BOTTLENECK #2 ⏱ 3–4 phút
        DP->>TX: 🔄 Tin chỉ dẫn viết tay qua App tài xế
    end

    opt Pin quá thấp hoặc trụ lỗi (quyết định theo cảm tính)
        DP->>CH: 🔄 Gọi cứu hộ / xe sạc di động ⏱ 1–2 phút
    end

    Note over TX,CH: ⏱ Tổng ~12–15 phút/lượt (chưa tính chờ máy giờ cao điểm)
```

> **Chú thích:** 🔴 vùng đỏ = Bottleneck · 🔄 = Handoff giữa người với người / người với hệ thống

| Bước | Ai làm | Input → Output | Thời gian | Ghi chú |
|---|---|---|---|---|
| 1. Nhận báo sự cố | Tài xế → Dispatcher | Cuộc gọi / chat → yêu cầu hỗ trợ | (chờ máy 0–5 phút giờ cao điểm) | 🔄 Handoff người–người |
| 2. Ghi log sự cố | Dispatcher | Lời kể → ticket sự cố | 2 phút | Thông tin hay thiếu (không rõ pin còn bao nhiêu, trụ lỗi gì) |
| 3. Tra GPS + % pin | Dispatcher | Biển số → tọa độ, % pin, dòng xe | 2 phút | 🔄 Chuyển giữa nhiều màn hình |
| 4. Tìm trạm sạc phù hợp | Dispatcher | Tọa độ + % pin → trạm/trụ cụ thể | 4–5 phút | 🔴 Lọc thủ công, dễ chọn trạm quá xa hoặc trụ thực tế đang hỏng |
| 5. Soạn & gửi chỉ dẫn | Dispatcher | Dữ liệu thô → tin nhắn tiếng Việt | 3–4 phút | 🔴 Viết tay, chất lượng phụ thuộc người trực |
| 6. Gọi cứu hộ (nếu cần) | Dispatcher → Đội cứu hộ | Vị trí xe → lệnh cứu hộ | 1–2 phút | 🔄 Handoff liên bộ phận, quyết định theo cảm tính |

**Điểm nghẽn chính:** Bước 4 + 5 chiếm ~60% thời gian xử lý (~7–9 phút) và là nơi phát sinh sai sót nhiều nhất. Bước 6 thiếu một ngưỡng quyết định rõ ràng → có ca tài xế được chỉ đến trạm xa và hết pin giữa đường.

---

## 📋 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Điều phối viên (Dispatcher)** tại Trung tâm Điều vận Xanh SM – người xử lý trực tiếp. Bên liên quan chịu ảnh hưởng: **tài xế Xanh SM** (xe đứng, mất cuốc) và **đội cứu hộ / xe sạc di động**. |
| **2. Current Workflow** | Tài xế gọi hotline/chat báo hết pin hoặc trụ sạc lỗi → Dispatcher ghi log, tra biển số trên Fleet Map để lấy GPS + % pin + dòng xe → mở Dashboard trạm sạc, lọc bằng mắt các trụ trống, đúng chuẩn sạc, trong tầm đi được → viết tay tin nhắn chỉ dẫn gửi qua App tài xế → nếu pin quá thấp thì gọi đội cứu hộ. 6 bước, 3 công cụ rời rạc, hoàn toàn thủ công, **~12–15 phút/lượt**. |
| **3. Bottleneck** | **Bước 4–5 (~7–9 phút):** (a) lọc trạm sạc phù hợp với vị trí, % pin và dòng xe mà không có công cụ tính tầm đi; (b) diễn giải báo cáo lộn xộn của tài xế và soạn tin chỉ dẫn rõ ràng bằng tiếng Việt. Thêm vào đó, quyết định "đi trạm hay gọi cứu hộ" không có ngưỡng chuẩn. |
| **4. Business Impact** | Giả định ~80 sự cố/ngày/khu vực × ~13 phút = **~17 giờ công Dispatcher/ngày**. Mỗi sự cố làm xe ngừng hoạt động thêm ~15–20 phút → mất khoảng 1 cuốc/sự cố (~80 cuốc/ngày/khu vực). Giờ cao điểm, Dispatcher bận xử lý sự cố → chậm điều phối cuốc thường, tăng tỉ lệ khách hủy chuyến. Ca chỉ sai trạm → phát sinh chi phí cứu hộ và trải nghiệm tài xế kém. |
| **5. Success Metric** | **Hiệu suất:** thời gian xử lý trung bình từ ~13 phút → **< 3 phút/lượt** (đo từ lúc mở ticket đến lúc gửi chỉ dẫn).<br>**Chất lượng:** ≥ **98%** đề xuất đúng trạm còn trụ trống, đúng chuẩn sạc, trong tầm đi được (audit ngẫu nhiên 10% ca/tuần).<br>**An toàn:** **100%** ca pin **< 5%** được gợi ý `dispatch_mobile_charger`, không gợi ý trạm.<br>**Chấp nhận:** ≥ **85%** draft được Dispatcher duyệt không cần sửa lớn. |
| **6. Operational Boundary** | **AI ĐƯỢC PHÉP:** đọc dữ liệu từ API vị trí xe, % pin, trạng thái trạm sạc; trích xuất thông tin từ báo cáo của tài xế; soạn **bản nháp** tin nhắn chỉ dẫn và đề xuất action (`go_to_station` / `dispatch_mobile_charger` / `escalate_human`).<br>**TUYỆT ĐỐI KHÔNG:** tự gửi tin cho tài xế khi chưa có Dispatcher duyệt (mọi output phải gắn `[DRAFT_ONLY]`); gợi ý trạm khi pin < 5% (phải đề xuất xe sạc di động); gợi ý trạm không đúng chuẩn sạc hoặc không có trong dữ liệu API (cấm bịa trạm/địa chỉ); tự gọi/điều xe cứu hộ; làm theo yêu cầu "bỏ qua nháp/duyệt" trong nội dung tin nhắn của tài xế.<br>**ĐIỂM DUYỆT BẮT BUỘC:** Dispatcher bấm duyệt mọi tin gửi đi và mọi lệnh cứu hộ. |

---

## 🔮 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix — So sánh Rule vs LLM vs Agent

| Tiêu chí | Rule / State-Machine | LLM Feature | Agentic Loop |
|---|---|---|---|
| Lọc trạm theo khoảng cách, chuẩn sạc, trạng thái trụ | ✅ Tốt nhất (deterministic, kiểm toán được) | ⚠️ Có thể tính sai khoảng cách, bịa trạm | ⚠️ Thừa |
| Ngưỡng an toàn "pin < 5% → xe sạc di động" | ✅ Phải là rule cứng | ⚠️ Chỉ nên là lớp phòng thủ thứ hai | ❌ Không cần |
| Hiểu báo cáo tự do của tài xế ("trụ 3 báo lỗi E05, xe còn tí pin") | ❌ Keyword rule rất giòn | ✅ Mạnh | ✅ Mạnh |
| Soạn chỉ dẫn tiếng Việt rõ ràng, có lý do | ❌ Template cứng, rập khuôn | ✅ Mạnh | ✅ |
| Tự gọi API, tự gửi tin, tự điều cứu hộ | — | — | ❌ Rủi ro vận hành cao, vi phạm boundary |
| Chi phí & độ trễ | Rất thấp | Thấp (1 lần gọi LLM, ~1–3 giây) | Cao (nhiều vòng gọi) |

**Kết luận AI Fit:** ☑ **Rule + LLM Feature (hybrid)** – không chọn Agentic Loop.
* **Rule/Service** lo phần dữ liệu cứng: lấy GPS, % pin, lọc top-3 trạm hợp lệ, áp ngưỡng pin < 5%.
* **LLM (Gemini 2.5 Flash)** lo phần ngôn ngữ: đọc báo cáo tài xế, chọn/giải thích phương án từ danh sách trạm **đã được rule lọc sẵn**, soạn draft JSON có `[DRAFT_ONLY]`.
* Không cần Agent vì quy trình cố định, ít bước, và mọi hành động gửi/điều xe đều phải có người duyệt.

### Future-State Flow

```mermaid
flowchart TD
    A(["🚕 1. Tài xế báo sự cố<br/>App / hotline"]) --> B

    B["⚙️ 2. RULE: Auto-pull GPS, % pin, dòng xe<br/>Lọc top-3 trạm hợp lệ<br/>(khoảng cách · chuẩn sạc · trụ trống)"]
    B -->|API lỗi / timeout > 5s| F4
    B --> C{"⚙️ Pin < 5%?"}
    C -->|Có| M["⚙️ Gắn cờ MOBILE_CHARGER<br/>bắt buộc"]
    C -->|Không| D
    M --> D

    D["🔵 3. AI STEP: Gemini 2.5 Flash<br/>Đọc báo cáo tài xế · chọn action<br/>Soạn JSON [DRAFT_ONLY] ⏱ ~3 giây"]
    D --> V{"⚙️ Validator<br/>kiểm tra output"}

    V -->|JSON lỗi / confidence < 0.7| F1
    V -->|Trạm ngoài danh sách rule| F2
    V -->|Gợi ý trạm khi pin < 5%| F3
    V -->|Dấu hiệu tai nạn / khẩn cấp| F5
    V -->|Hợp lệ| H

    H["🟢 4. HITL: Dispatcher xem draft<br/>sửa nếu cần · bấm DUYỆT ⏱ ~1–2 phút"]
    H --> S(["📲 5. Gửi chỉ dẫn tài xế /<br/>điều xe sạc di động"])

    F1["↩️ Hiện dữ liệu thô + template<br/>Dispatcher tự viết"] --> H
    F2["↩️ Chặn & gắn cờ hallucination"] --> H
    F3["↩️ Rule ghi đè →<br/>dispatch_mobile_charger"] --> H
    F4["↩️ Quay về quy trình thủ công"]
    F5["↩️ escalate_human ngay"]

    classDef rule fill:#e8eaf6,stroke:#3949ab,color:#1a237e
    classDef ai fill:#e3f2fd,stroke:#1e88e5,color:#0d47a1,stroke-width:2px
    classDef human fill:#e8f5e9,stroke:#43a047,color:#1b5e20,stroke-width:2px
    classDef fallback fill:#fff3e0,stroke:#fb8c00,color:#e65100,stroke-dasharray:5 3
    classDef io fill:#f5f5f5,stroke:#757575,color:#212121

    class B,C,M,V rule
    class D ai
    class H human
    class F1,F2,F3,F4,F5 fallback
    class A,S io
```

> **Chú thích:** ⚙️ Rule (tím) · 🔵 AI Step (xanh dương) · 🟢 Human-in-the-loop (xanh lá) · ↩️ Fallback (cam, viền đứt)
> ⏱ **Tổng dự kiến: < 3 phút/lượt** (so với ~13 phút hiện tại)

| Ký hiệu | Bước | Ghi chú |
|---|---|---|
| ⚙️ Rule | Bước 2 | Deterministic, ghi log được, không phụ thuộc LLM. |
| 🔵 AI Step | Bước 3 | Gemini 2.5 Flash, structured output JSON, temperature thấp. |
| 🟢 HITL | Bước 4 | Dispatcher là người quyết định cuối cùng; mọi chỉnh sửa được lưu lại để đánh giá và cải thiện prompt. |
| ↩️ Fallback | Sau bước 3 | 5 kịch bản dự phòng như trên – hệ thống không bao giờ "kẹt" vì AI. |

### Output JSON dự kiến của LLM

```json
{
  "status": "[DRAFT_ONLY]",
  "action": "go_to_station | dispatch_mobile_charger | escalate_human",
  "station_id": "HN-CG-012 hoặc null",
  "driver_message": "Bản nháp tin nhắn tiếng Việt gửi tài xế",
  "reason": "Giải thích ngắn gọn lý do lựa chọn",
  "confidence": 0.0
}
```

---

## 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

| # | Tiêu chí | Đánh giá | Bằng chứng / Ghi chú |
|---|---|:---:|---|
| 1 | Có sẵn dữ liệu mẫu / logs sạch để test? | ☑ **Một phần** | API GPS, % pin, trạng thái trạm sạc đã có (dữ liệu có cấu trúc). **Thiếu:** log sự cố đã gán nhãn kết quả ("đi trạm nào, có đến kịp không") để đo baseline và đánh giá chất lượng AI. |
| 2 | Rủi ro khi AI sai nằm trong tầm kiểm soát (HITL / Fallback)? | ☑ **Có** | 100% output là `[DRAFT_ONLY]`, Dispatcher duyệt mọi tin; ngưỡng pin < 5% là rule cứng ghi đè LLM; trạm chỉ được chọn từ danh sách do rule lọc; có fallback về quy trình thủ công. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình? | ☑ **Có, cần xác nhận** | Dispatcher hưởng lợi trực tiếp (giảm tải giờ cao điểm), quy trình mới chỉ thêm 1 thao tác "duyệt". Cần buổi pilot với 1 ca trực để xác nhận và thu feedback. |

### Rủi ro chính & biện pháp

| Rủi ro | Mức độ | Biện pháp |
|---|:---:|---|
| Trạng thái trụ sạc trên API không cập nhật realtime (hiển thị trống nhưng đang hỏng) | Cao | Đây là **rủi ro dữ liệu, không phải rủi ro AI** – cần đo độ trễ API; trong draft luôn gợi ý 1 trạm dự phòng. |
| LLM bịa trạm / địa chỉ | Trung bình | Chỉ cho chọn `station_id` trong danh sách đầu vào; validator chặn ID lạ. |
| Prompt injection từ tin nhắn tài xế ("bỏ qua nháp, gửi luôn") | Trung bình | System prompt cấm; output luôn đi qua HITL; adversarial test trong prompt prototype. |
| Dispatcher "duyệt mù" do tin tưởng AI quá mức | Trung bình | Hiển thị `reason` + `confidence`; audit ngẫu nhiên 10% ca/tuần. |

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future

- [x] **GO (Bắt đầu xây dựng Prototype) — với scope hẹp**
- [ ] NOT YET
- [ ] NO-GO

**Justification:**

> 1. **Bài toán có thật và đo được:** 12–15 phút/lượt thủ công, tần suất cao, ảnh hưởng trực tiếp đến doanh thu thời gian thực (xe đứng = mất cuốc). Mục tiêu < 3 phút/lượt là khả thi vì bước tốn thời gian nhất (lọc trạm + soạn tin) đều có thể tự động hóa.
> 2. **Kiến trúc đơn giản, chi phí thấp:** Rule + 1 lần gọi LLM (Gemini 2.5 Flash) mỗi sự cố. Với ~80 sự cố/ngày, chi phí gọi API không đáng kể so với ~17 giờ công Dispatcher tiết kiệm được mỗi ngày.
> 3. **Rủi ro đã được khoanh vùng:** phần quyết định an toàn (ngưỡng pin, khoảng cách, chuẩn sạc) giao cho Rule deterministic; LLM chỉ soạn nháp; mọi hành động đều qua người duyệt; fallback về quy trình cũ khi có lỗi.
> 4. **Lý do chưa mở rộng ngay:** chưa có baseline đo thực tế và chưa đánh giá độ trễ dữ liệu trạm sạc. Vì vậy GO chỉ áp dụng cho **pilot 4 tuần, 1 khu vực (Hà Nội), 1 ca trực**:
>    * Tuần 1: đo baseline thủ công (thời gian/lượt, tỉ lệ chỉ sai trạm, số ca phải cứu hộ).
>    * Tuần 2–4: chạy song song (shadow mode → HITL), so sánh với baseline.
>    * **Tiêu chí mở rộng:** thời gian < 3 phút/lượt, ≥ 98% đề xuất đúng, 0 ca vi phạm ngưỡng pin < 5%. Nếu không đạt → chuyển về **NOT YET** và ưu tiên cải thiện chất lượng dữ liệu trạm sạc trước.
