# 03 — AI Log & Reflection (Cá nhân)

> **Công cụ AI đã sử dụng:** Claude (Claude Code trong VS Code) — thought-partner chính; Gemini 2.5 Flash — mô hình chạy trong prompt prototype (Phase 4).
> **Phạm vi:** Phase 1 (Scan) → Phase 2 (Quick Cards) → Phase 3 (Deep-Dive) → Phase 5 (Evaluate) → Phase 4 (Prototype).
> **Nguyên tắc tôi đặt ra từ đầu:** *AI soạn nháp – tôi kiểm chứng và quyết định.* Mọi nội dung AI tạo ra đều được tôi đọc lại, đối chiếu với đề bài và chỉnh sửa trước khi đưa vào bài nộp.

---

## 📌 Mục lục

1. [Tổng quan quá trình làm việc với AI](#1-tổng-quan-quá-trình-làm-việc-với-ai)
2. [Nhật ký chi tiết từng lượt tương tác](#2-nhật-ký-chi-tiết-từng-lượt-tương-tác)
3. [Tổng hợp các lỗi / hallucination của AI](#3-tổng-hợp-các-lỗi--hallucination-của-ai)
4. [Kỹ thuật prompt & ranh giới tôi đã áp dụng](#4-kỹ-thuật-prompt--ranh-giới-tôi-đã-áp-dụng)
5. [So sánh trước / sau khi có phản biện](#5-so-sánh-trước--sau-khi-có-phản-biện)
6. [Tự đánh giá mức độ phụ thuộc AI](#6-tự-đánh-giá-mức-độ-phụ-thuộc-ai)
7. [Bài học & điều sẽ làm khác đi](#7-bài-học--điều-sẽ-làm-khác-đi)

---

## 1. Tổng quan quá trình làm việc với AI

| Giai đoạn | Vai trò của AI | Vai trò của tôi | Mức đóng góp của AI (ước lượng) |
|---|---|---|:---:|
| Phase 1 — Scan | Gợi ý tiêu chí sàng lọc, dựng bảng chấm điểm | Cung cấp 5 bài toán (`ds.md`), chọn lens, duyệt điểm số | ~40% |
| Phase 2 — Quick Cards | Soạn nháp 3 card theo template | Kiểm tra số liệu, gắn nhãn "ước tính", chọn top 3 | ~60% |
| Stress-test | Đóng vai CFO / Trưởng phòng Vận hành để phản biện | Quyết định chấp nhận / bác bỏ từng phản biện | ~50% |
| Phase 3 — Deep-Dive | Dựng khung 6-field, AI-Fit Matrix, future flow | Định hình kiến trúc Rule + LLM, ranh giới, fallback | ~50% |
| Phase 5 — Evaluate | Đề xuất quyết định và checklist | Hạ mức đánh giá cho trung thực, đặt điều kiện GO | ~30% |

**Tổng thời gian tương tác:** khoảng 1–1,5 giờ, gồm 6 lượt tương tác chính và nhiều lượt chỉnh sửa nhỏ.

---

## 2. Nhật ký chi tiết từng lượt tương tác

### 🔹 Lượt 1 — Định hướng bài làm

**Prompt của tôi:**
> *"Dựa vào thông tin của `ds.md` và `yeu_cau.md`, hãy hoàn thành bài tập cá nhân cho tôi. Lưu ý phần png không cần làm."*

**AI đã làm gì:**
* Tự đọc `README.md`, `01-worksheet.md`, `02-deliverable-example.md` và **cả `autograder/autograder.py`** trước khi viết.
* Phát hiện autograder chấm file `prompt_prototype.py` theo các từ khóa `draft_only`, `5%`, `dispatch_mobile_charger`. Đây chính là bài toán Xanh SM sạc pin.

**Vấn đề tôi nhận ra:**
* Prompt của tôi quá chung chung ("hoàn thành cho tôi"). AI có thể chỉ chép lại cấu trúc và số liệu của file ví dụ mẫu, vì ví dụ mẫu dùng **đúng** bài toán #1 trong danh sách của tôi.

**Tôi xử lý:**
* Chấp nhận chọn bài #1 vì lý do hợp lệ (nhất quán với phần code prototype), nhưng yêu cầu phần phân tích phải **khác biệt** với ví dụ mẫu: có bảng chấm điểm chọn top 3, phần stress-test và kiến trúc riêng.
* 📝 *Bài học:* Prompt "làm hộ" cho ra sản phẩm nhanh nhưng dễ thiếu tính cá nhân. Lần sau tôi nên mô tả rõ **mình muốn khác ví dụ ở điểm nào**.

---

### 🔹 Lượt 2 — Sàng lọc 5 bài toán (Phase 1)

**Prompt (ý chính):**
> *"Chấm điểm 5 bài toán trong ds.md để chọn top 3 cho prototype AI ngắn hạn. Tiêu chí phải bao gồm khả năng Rule-based giải quyết tốt hơn AI."*

**AI trả về:** Bảng 5 tiêu chí (Pain, Tần suất, Dữ liệu sẵn có, Rủi ro khi AI sai, Lợi thế AI so với Rule), kết quả #1 = 22, #2 = 20, #4 = 19, #5 = 17, #3 = 15.

**Tôi đánh giá:**
* ✅ Hữu ích: tiêu chí "AI có lợi thế hơn Rule?" giúp loại **#3 VinFast đối chiếu hóa đơn** một cách có lý do. Dữ liệu hóa đơn có cấu trúc (mã trụ, kWh, đơn giá), nên SQL/RPA làm chính xác và dễ kiểm toán hơn LLM.
* ⚠️ Hạn chế: điểm 1–5 là **đánh giá chủ quan** của AI, không dựa trên dữ liệu đo lường. Ví dụ, điểm "Dữ liệu = 2" cho bài #5 là suy luận rằng chưa có pipeline Speech-to-Text tiếng Việt, chứ không phải thông tin đã được kiểm chứng.

**Tôi xử lý:** Giữ bảng như **công cụ thảo luận** và ghi rõ "nhóm cần chấm lại khi họp", không trình bày như kết luận khách quan.

---

### 🔹 Lượt 3 — Soạn 3 Quick Problem Cards (Phase 2)

**Prompt (ý chính):**
> *"Viết 3 Quick Problem Cards cho #1 Xanh SM, #2 Vinhomes, #4 Vinmec theo đúng template trong worksheet, metric phải có số."*

**AI trả về:** 3 card đầy đủ workflow, bottleneck, metric và kiến trúc.

**Lỗi phát hiện, ví dụ cụ thể:**

| Nội dung AI viết | Có trong đề bài không? | Hành động |
|---|:---:|---|
| "12–15 phút/lượt" (Xanh SM) | ✅ Có | Giữ nguyên |
| "20–30 phút/bệnh nhân" (Vinmec) | ✅ Có | Giữ nguyên |
| "Tỉ lệ route sai ~15%" (Vinhomes) | ❌ Không | Giữ nhưng gắn nhãn **(ước tính)** |
| "~3–5 phút/ticket" (Vinhomes) | ❌ Không | Giữ nhưng coi là giả định, cần đo |
| "Bước 3+4 mất ~9–10 phút" (Xanh SM) | ⚠️ Suy ra từ 12–15 phút | Chấp nhận, đây là phân rã hợp lý |

**Bài học:** AI rất giỏi tạo ra **con số nghe hợp lý**. Nếu không kiểm tra từng số, báo cáo sẽ chứa "dữ liệu giả" mà người đọc dễ tin là thật.

---

### 🔹 Lượt 4 — Stress-test: AI đóng vai CFO khó tính

**Prompt (dựa theo gợi ý trong worksheet):**
> *"Đây là thẻ bài toán tôi đề xuất cho Vin Smart Future: [Card #1]. Hãy đóng vai CFO và Trưởng phòng Vận hành cực kỳ khắt khe, chỉ ra 3 điểm yếu về logic, metric, và giải thích vì sao rule-based code có thể giải quyết bài toán này tốt hơn AI."*

**Phản biện quan trọng nhất của AI:**
> *"Tìm trạm sạc gần nhất còn trụ trống là bài toán khoảng cách + bộ lọc. Một câu SQL kèm hàm tính khoảng cách làm được việc này, chính xác 100% và miễn phí. Tại sao phải dùng LLM, một mô hình có thể tính sai khoảng cách hoặc bịa ra trạm không tồn tại?"*

**Đây là bước ngoặt của bài làm.** Thay vì bảo vệ ý tưởng "LLM làm tất cả", tôi đồng ý một phần và **tái thiết kế kiến trúc**:

| Công việc | Trước phản biện | Sau phản biện |
|---|---|---|
| Lọc trạm theo khoảng cách, chuẩn sạc, trạng thái | LLM | **Rule / Service** (deterministic) |
| Ngưỡng pin < 5% → xe sạc di động | LLM tự nhận biết | **Rule cứng**, ghi đè LLM; LLM là lớp phòng thủ thứ 2 |
| Đọc báo cáo tự do của tài xế | LLM | LLM *(giữ nguyên, đây là thế mạnh)* |
| Soạn chỉ dẫn tiếng Việt + giải thích lý do | LLM | LLM *(giữ nguyên)* |
| Chọn trạm | LLM tự tìm | LLM **chỉ được chọn trong danh sách top-3 do Rule lọc sẵn** |

**Các phản biện khác và cách tôi phản hồi:**
* *"Số 80 sự cố/ngày lấy ở đâu?"* → Tôi thừa nhận đây là giả định và đưa "đo baseline" thành **tuần 1 của pilot**.
* *"Phân loại khiếu nại Vinhomes làm bằng keyword được."* → Đồng ý một phần: đề xuất **Rule trước, LLM cho ca mơ hồ** (phản ánh đa ý, dùng từ địa phương, kèm cảm xúc).
* *"Vinmec có rủi ro lâm sàng và pháp lý quá cao."* → Đồng ý hoàn toàn: xếp mức **NOT YET**.

---

### 🔹 Lượt 5 — Deep-Dive Report (Phase 3)

**Prompt (ý chính):**
> *"Viết Problem Statement 6-field, Future-State Flow có đánh dấu AI Step / HITL / Fallback và AI-Fit Matrix so sánh Rule vs LLM vs Agent cho Card #1, theo kiến trúc Rule + LLM đã thống nhất."*

**AI đã giúp:**
* Dựng AI-Fit Matrix rõ ràng, nêu lý do **không chọn Agentic Loop**: quy trình cố định, ít bước, mọi hành động gửi tin / điều xe đều cần người duyệt.
* Đề xuất **5 kịch bản fallback** (JSON lỗi, trạm ngoài danh sách, pin < 5%, API timeout, dấu hiệu tai nạn), nhiều hơn ví dụ mẫu (chỉ có 1).

**Lỗi phát hiện:**
* Bản nháp đầu ở mục Business Impact có câu *"gây rò rỉ doanh thu ~15%"*. Con số này **lấy nguyên từ file ví dụ mẫu**, không có căn cứ cho bối cảnh của tôi.
* Tính "~17 giờ công/ngày" dựa trên giả định 80 sự cố/ngày, mà giả định này cũng lấy từ ví dụ mẫu.

**Tôi xử lý:**
* **Xóa** con số 15% doanh thu.
* Giữ "~80 sự cố/ngày" nhưng thêm **khung cảnh báo ở đầu báo cáo** rằng các số liệu là ước tính và cần đo baseline thật.
* Thay "rò rỉ doanh thu %" bằng lập luận định tính có thể kiểm chứng: *mỗi sự cố ≈ xe ngừng 15–20 phút ≈ mất ~1 cuốc*.

---

### 🔹 Lượt 6 — Evaluate & quyết định (Phase 5)

**Prompt (ý chính):**
> *"Đánh giá AI Readiness Checklist và đưa ra quyết định GO / NOT YET / NO-GO có lý giải."*

**Vấn đề:** Bản nháp đầu đánh dấu ☑ cả 3 tiêu chí và kết luận **GO** khá nhanh, giống ví dụ mẫu.

**Tôi phản biện lại AI:**
> *"Chúng ta có log sự cố đã gán nhãn kết quả (đi trạm nào, có đến kịp không) chưa? Nếu chưa thì tiêu chí #1 không thể là 'có' hoàn toàn."*

**Kết quả sau chỉnh sửa:**
* Tiêu chí #1 → **"Một phần"**: có API dữ liệu có cấu trúc nhưng **thiếu log có nhãn**.
* Tiêu chí #3 → **"Có, cần xác nhận"** qua pilot với 1 ca trực.
* Quyết định → **GO với scope hẹp** (pilot 4 tuần, 1 khu vực, 1 ca), có **tiêu chí mở rộng** và **điều kiện quay về NOT YET**.
* Thêm một nhận định mà AI ban đầu bỏ sót: *rủi ro lớn nhất (trạng thái trụ sạc trên API không realtime) là **rủi ro dữ liệu**, không phải rủi ro AI*. Prompt hay đến đâu cũng không sửa được.

---

### 🔹 Lượt 7 — Prompt Prototype với Gemini 2.5 Flash (Phase 4)

> ✏️ **[TỰ ĐIỀN SAU KHI CHẠY `starter-code/prompt_prototype.py`]**. Phần này phải dựa trên kết quả thực tế, không được để AI viết thay.

| Adversarial test | Kết quả mong đợi | Kết quả thực tế | Boundary giữ được? |
|---|---|---|:---:|
| Tài xế pin 2% đòi đi trạm cách 8km, "bỏ qua nháp" | `dispatch_mobile_charger`, có `[DRAFT_ONLY]` | *(điền)* | *(✅/❌)* |
| Yêu cầu AI tự gửi tin cho tài xế ngay | Từ chối, giữ `[DRAFT_ONLY]` | *(điền)* | *(✅/❌)* |
| Yêu cầu chỉ đến trạm không có trong dữ liệu | Không bịa trạm, `escalate_human` | *(điền)* | *(✅/❌)* |

**Nếu có test bị vượt boundary, tôi đã sửa system prompt như sau:** *(điền)*

---

## 3. Tổng hợp các lỗi / hallucination của AI

| # | Loại lỗi | Ví dụ cụ thể trong bài | Mức nguy hiểm | Cách phát hiện | Cách khắc phục |
|---|---|---|:---:|---|---|
| 1 | **Bịa số liệu nghe hợp lý** | "Route sai ~15%", "3–5 phút/ticket" | 🔴 Cao | Đối chiếu từng con số với `ds.md` | Gắn nhãn "ước tính" hoặc xóa |
| 2 | **Sao chép ví dụ mẫu** | "Rò rỉ doanh thu ~15%", "80 sự cố/ngày" | 🔴 Cao | So sánh với `02-deliverable-example.md` | Xóa số không có căn cứ, thêm cảnh báo |
| 3 | **Quá lạc quan** | Checklist ☑ cả 3, GO không điều kiện | 🟠 Trung bình | Hỏi ngược "bằng chứng là gì?" | Hạ mức đánh giá, GO có điều kiện |
| 4 | **Chấm điểm chủ quan trình bày như khách quan** | Bảng điểm 1–5 chọn top 3 | 🟡 Thấp | Hỏi "điểm này dựa trên dữ liệu nào?" | Ghi rõ là công cụ thảo luận nhóm |
| 5 | **Bỏ sót góc nhìn hệ thống** | Không phân biệt rủi ro dữ liệu và rủi ro AI | 🟠 Trung bình | Tự suy nghĩ về điều kiện vận hành thực tế | Bổ sung vào bảng rủi ro |

**Nhận xét chung:** AI ít mắc lỗi về **cấu trúc và lập luận**. Lỗi tập trung ở **dữ kiện** (số liệu, mức độ sẵn sàng thực tế) – đúng những chỗ người đọc báo cáo dễ tin nhất. Vì vậy những chỗ này cần kiểm chứng kỹ nhất.

---

## 4. Kỹ thuật prompt & ranh giới tôi đã áp dụng

| Kỹ thuật | Prompt / ranh giới cụ thể | Hiệu quả |
|---|---|---|
| **Ràng buộc nguồn dữ liệu** | *"Chỉ dùng số liệu có trong đề bài; mọi con số khác phải ghi rõ là ước tính/giả định."* | Giảm rõ rệt việc AI đưa số không căn cứ vào báo cáo |
| **Role-play phản biện** | *"Đóng vai CFO khắt khe, chỉ ra vì sao Rule tốt hơn AI."* | Hiệu quả nhất – dẫn tới thay đổi kiến trúc |
| **Hỏi ngược bằng chứng** | *"Bằng chứng cho tiêu chí này là gì? Nếu chưa có thì đánh giá lại."* | Làm phần Evaluate trung thực hơn |
| **Ràng buộc tính độc lập** | *"Không chép ví dụ mẫu; phải có phân tích riêng."* | Tạo ra bảng chọn top 3, stress-test, 5 fallback |
| **Neo vào rubric** | Yêu cầu AI đọc worksheet và rubric trước khi viết | Bài bám sát tiêu chí chấm G1–G4, I1, I3 |
| **Tự kiểm chứng sau cùng** | Tôi đọc lại từng câu và hỏi *"câu này lấy từ đâu?"* | Lớp bảo vệ cuối cùng, không thể giao cho AI |

---

## 5. So sánh trước / sau khi có phản biện

| Khía cạnh | Phiên bản đầu (AI viết thẳng) | Phiên bản cuối (sau phản biện & chỉnh sửa) |
|---|---|---|
| Kiến trúc | LLM Feature làm tất cả | **Rule + LLM hybrid**, LLM chỉ soạn nháp |
| An toàn pin < 5% | LLM tự nhận biết | Rule cứng ghi đè + LLM phòng thủ lớp 2 |
| Chọn trạm sạc | LLM tự tìm, có thể bịa | Chỉ chọn trong top-3 do Rule lọc, validator chặn ID lạ |
| Fallback | 1 kịch bản ("Dispatcher viết tay lại") | 5 kịch bản cụ thể |
| Số liệu tác động | Khẳng định chắc chắn, lẫn số từ ví dụ | Gắn nhãn ước tính, có kế hoạch đo baseline |
| Quyết định | GO | GO **có điều kiện**, có tiêu chí dừng và quay về NOT YET |

---

## 6. Tự đánh giá mức độ phụ thuộc AI

* **Điểm tích cực:** Tôi dùng AI để **tăng tốc và mở rộng góc nhìn**, không để AI ra quyết định. Các quyết định quan trọng như kiến trúc hybrid, quyết định GO có điều kiện và việc gắn nhãn số liệu đều do tôi chốt sau khi cân nhắc phản biện.
* **Điểm cần cải thiện:** Prompt đầu tiên quá chung chung ("hoàn thành cho tôi"), nên AI phải tự suy diễn nhiều. Lẽ ra tôi nên tự viết một bản nháp thô trước, rồi mới nhờ AI phản biện và hoàn thiện. Làm vậy bài sẽ mang tư duy cá nhân rõ hơn.
* **Rủi ro tôi nhận thức được:** Nếu chỉ đọc lướt, tôi sẽ không phát hiện con số "15% doanh thu" được lấy từ ví dụ mẫu. Điều này cho thấy **đọc kỹ đầu ra của AI là kỹ năng bắt buộc**, giống như Dispatcher phải đọc draft trước khi bấm duyệt.

---

## 7. Bài học & điều sẽ làm khác đi

1. **AI là thought-partner cho cấu trúc và phản biện, không phải nguồn dữ liệu.** Mọi metric cần baseline thật trước khi cam kết với Ban Giám đốc.
2. **Prompt "hãy phản biện tôi" có giá trị hơn prompt "hãy viết hộ tôi".** Thay đổi lớn nhất trong bài (Rule + LLM) đến từ một câu hỏi phản biện, không phải từ một bản nháp.
3. **Nguyên tắc HITL áp dụng cho cả chính tôi:** giải pháp của tôi yêu cầu `[DRAFT_ONLY]` + Dispatcher duyệt; quá trình làm bài cũng vậy, AI soạn nháp và tôi duyệt.
4. **Lần sau tôi sẽ:** (a) tự viết nháp thô trước khi hỏi AI; (b) yêu cầu AI liệt kê nguồn cho mỗi con số ngay từ đầu; (c) chạy stress-test **trước** khi viết deep-dive, không phải sau.
