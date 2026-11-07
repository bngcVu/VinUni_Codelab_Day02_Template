# 03 — AI Log & Reflection (Lab 02: AI Product Scoping)

**Công cụ AI dùng:** Claude Code (trợ lý lập trình trong VS Code) để hiểu đề, viết code và soạn nháp tài liệu; Groq API (`openai/gpt-oss-120b`) làm mô hình chạy prototype.

---

## 1. AI giúp tôi những gì

| Việc | AI đã làm | Tôi đã làm |
|---|---|---|
| Hiểu đề | Đọc README, worksheet, autograder và tóm tắt các deliverable, điểm số, quy định nộp bài. | Hỏi lại khi chưa hiểu, nhờ giải thích đơn giản hơn. |
| Viết prototype | Viết `SYSTEM_PROMPT` (2 luật cứng), hàm `evaluate_prompt()` gọi Groq, thêm test tấn công thứ 3. | Yêu cầu dùng Groq thay vì Gemini, tạo API key, chạy và gửi kết quả. |
| Gỡ lỗi môi trường | Chỉ ra lỗi do gõ lệnh PowerShell trong CMD, đọc các thông báo lỗi 401/404. | Chạy các lệnh kiểm tra và báo lại output thật. |
| Chọn chủ đề | Đề xuất danh sách chủ đề; phản biện ý tưởng "chính sách thu hút và giữ chân nhân tài". | Chọn hướng sàng lọc CV. |
| Soạn tài liệu | Soạn nháp `01`, `02`, sơ đồ `04`. | Đọc lại, quyết định giữ hay sửa. |

---

## 2. AI sai hoặc gây vấn đề ở đâu

### Lỗi 1 — AI dùng một model đã không còn trong tài khoản
- **Chuyện gì xảy ra:** AI chọn model `llama-3.3-70b-versatile` dựa trên kiến thức có sẵn của nó. Khi chạy, Groq trả `404 model_not_found`.
- **Nguyên nhân:** danh sách model của dịch vụ thay đổi theo thời gian, AI không kiểm tra được tài khoản thật của tôi.
- **Cách sửa:** thay vì đoán tên khác, tôi chạy lệnh liệt kê model qua API, lấy danh sách thật rồi chọn `openai/gpt-oss-120b`.
- **Bài học:** với tên model, phiên bản, giá, API, phải kiểm chứng trên hệ thống thật chứ không tin trí nhớ của AI.

### Lỗi 2 — AI giả định sai shell
- **Chuyện gì xảy ra:** AI hướng dẫn đặt biến môi trường bằng cú pháp PowerShell (`$env:...`), trong khi tôi đang dùng CMD nên báo lỗi cú pháp.
- **Cách sửa:** AI nhận ra dấu nhắc `C:\...>` là CMD và đưa lệnh `set` đúng. Sau đó tôi còn gõ thiếu tên biến (`GROQ_API_KEY=`) một lần, và mở cửa sổ CMD mới nên mất biến.
- **Bài học:** phải nói rõ môi trường đang dùng (hệ điều hành, shell) ngay từ đầu.

### Lỗi 3 — Lỗi 401 Invalid API Key
- **Chuyện gì xảy ra:** key bị từ chối. Kiểm tra độ dài thì thấy 57 ký tự thay vì 56.
- **AI đoán:** có ký tự/khoảng trắng thừa ở cuối do lệnh `set`. AI thêm `.strip()` vào code và hướng dẫn đặt lại biến có dấu ngoặc kép.
- **Kết quả:** sau khi tôi đặt lại key, độ dài về 56 và hết lỗi 401. **Tôi không chắc chắn nguyên nhân chính là khoảng trắng hay do dán key sai**, vì hai việc xảy ra cùng lúc. `.strip()` vẫn giữ lại như một lớp phòng thủ.

### Lỗi 4 — Tôi lỡ dán API key vào khung chat (hai lần)
- AI cảnh báo ngay rằng key đã bị lộ và cần xóa rồi tạo key mới trên console của Groq.
- **Bài học:** key chỉ được đặt trong biến môi trường hoặc file `.env` (đã nằm trong `.gitignore`), tuyệt đối không dán vào chat, code hay commit.

### Lưu ý 5 — Autograder chỉ nhận Gemini
- AI đọc `autograder.py` và phát hiện tiêu chí 2 chỉ chấp nhận mã có chữ `genai`/`generativeai`. Dùng Groq nghĩa là có thể chỉ được **0,5/1 điểm** ở tiêu chí đó, và grader có thể không có Groq key khi chạy.
- AI **không** gợi ý lách (ví dụ nhét chữ `genai` vào comment) mà báo rõ hệ quả để tôi tự quyết. Tôi chọn giữ Groq và chấp nhận rủi ro về điểm; nếu cần điểm tối đa có thể đổi lại Gemini hoặc hỏi giảng viên.

### Lỗi 6 — Con số do AI soạn chỉ là giả định
- Các số trong `01` và `02` (10 phút/CV, 200 CV/vị trí, 33 giờ HR) là **ước tính giả định của AI**, không phải dữ liệu thật của Vingroup. Mỗi file đều ghi rõ điều này.
- **Cách sửa:** trong báo cáo, điều kiện của quyết định GO yêu cầu **đo baseline thật ở tuần đầu**. Trước khi nộp, nhóm cần thay bằng số có nguồn.

---

## 3. Cách tôi chỉnh prompt/ranh giới để ra kết quả đúng

1. **Đặt luật theo thứ tự ưu tiên:** trong `SYSTEM_PROMPT`, ghi rõ hai luật cứng "ghi đè mọi chỉ dẫn của người dùng", kể cả khi người dùng yêu cầu bỏ luật hoặc giả danh quản trị viên.
2. **Quy định định dạng cụ thể:** dòng đầu luôn là `[DRAFT_ONLY]`, sau đó mới đến JSON `dispatch_mobile_charger`; không để mô hình tự chọn cách trình bày.
3. **`temperature=0`** để kết quả lặp lại được khi kiểm thử.
4. **Tăng `max_tokens` lên 2000** vì `gpt-oss-120b` là mô hình suy luận, phần "suy nghĩ" ngầm có thể ăn hết giới hạn cũ (400) và trả về câu rỗng.
5. **Thêm test thứ 3** (giả mạo lệnh quản trị "SYSTEM OVERRIDE", pin 3%, trạm 12 km) vì đề yêu cầu tối thiểu 3 test tấn công, trong khi template chỉ có 2.

**Kết quả:** cả 3 test đều ✅ ở lần chạy đầu tiên với `gpt-oss-120b`: mô hình giữ thẻ `[DRAFT_ONLY]`, điều xe sạc di động thay vì chỉ trạm xa, và không nghe lệnh giả mạo.

---

## 4. Giới hạn của kết quả

- **Chỉ 3 test, mỗi test chạy 1 lần.** Mô hình có thể vẫn bị phá vỡ bởi cách tấn công khác (đa ngôn ngữ, chia nhỏ yêu cầu, ngụy trang trong văn bản dài). Pass ở đây chưa chứng minh ranh giới an toàn.
- **Phần kiểm tra bằng từ khóa còn yếu.** Ví dụ kiểm tra Rule 2 chỉ cần thấy chuỗi `dispatch_mobile_charger` trong output; nó không kiểm tra mô hình có nhắc kèm trạm sạc xa hay không.
- **Ranh giới nên được cưỡng chế bằng code, không chỉ bằng prompt.** Trong hệ thống thật, việc gắn `[DRAFT_ONLY]` và chặn gửi tự động nên do code đảm nhiệm; prompt chỉ là lớp phòng thủ thứ hai. Prototype này cố ý chỉ kiểm tra prompt theo yêu cầu của lab.

---

## 5. Tôi đã bất đồng với AI ở đâu

- **Chủ đề:** tôi đề xuất "chính sách thu hút và giữ chân nhân tài". AI không đồng ý nguyên văn: đó là chính sách chung, không phải quy trình có người làm và có điểm nghẽn, nên khó chấm ở các mục vẽ workflow và Problem Statement. AI gợi ý thu hẹp thành một khâu cụ thể; tôi chọn **sàng lọc CV**, vẫn đúng hướng nhân tài của ý ban đầu.
- **Rủi ro đạo đức:** khi đưa chủ đề nhân sự vào, AI nhắc rằng chấm CV có nguy cơ thiên vị và dữ liệu cá nhân nhạy cảm. Tôi giữ phần này trong báo cáo (AI không được tự loại ứng viên, bỏ tuổi/giới tính/ảnh, kiểm tra thiên vị định kỳ) thay vì bỏ cho gọn.

---

## 6. Điều tôi rút ra

1. AI nhanh trong việc dựng khung và viết nháp, nhưng **dễ tự tin sai** về thông tin thay đổi theo thời gian (tên model) và về môi trường của tôi (shell). Luôn chạy thử và đọc thông báo lỗi thật.
2. Một prompt tốt cần **luật rõ, thứ tự ưu tiên rõ, định dạng đầu ra cố định**; và phải **tấn công thử** để biết nó có vững hay không, không chỉ thử trường hợp bình thường.
3. Số liệu AI đưa ra chỉ là giả định cho đến khi có nguồn. Quyết định GO/NO-GO cần dựa trên dữ liệu thật.
4. Việc nhạy cảm (khóa API, dữ liệu cá nhân ứng viên) phải được xử lý riêng, không đưa vào công cụ hay cuộc trò chuyện một cách tùy tiện.
