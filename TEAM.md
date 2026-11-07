# 👥 TEAM — Tổng hợp bài làm nhóm (Lab 02: AI Product Scoping — Vin Smart Future)

> **Trưởng nhóm:** Bùi Ngọc Vũ
> **Vai trò của file này:** Trưởng nhóm tổng hợp bài làm cá nhân của các thành viên, ghi lại quá trình chọn bài và các file `.md` được merge vào `main` (theo [README.md](README.md), mục 3–4). Branch `main` là căn cứ duy nhất để tính điểm báo cáo chung của nhóm.

---

## 1. Thành viên nhóm

| # | Họ và tên | Vai trò | Branch cá nhân |
|:-:|---|---|---|
| 1 | **Bùi Ngọc Vũ** | 👑 **Trưởng nhóm** | `2A202603463-BuiNgocVu` |
| 2 | Nguyễn Thế Trường | Thành viên | `2A202603393-NguyenTheTruong` |
| 3 | Dương Tuấn Hưng | Thành viên | `2A202603416-DuongTuanHung` |
| 4 | Tô Minh Thành | Thành viên | `2A202603439-ToMinhThanh` |

---

## 2. Quá trình chọn bài cho branch `main`

**Tiêu chí chọn** (bám theo rubric điểm nhóm):

| Gate | Điểm | Tiêu chí đánh giá khi so sánh các bài |
|---|:-:|---|
| G1. Workflow Mapping | 20 | Sơ đồ rõ bước, handoff, thời gian, bottleneck |
| G2. Problem Statement | 20 | Đủ 6 field, metric có số, ranh giới rõ ràng |
| G3. AI Fit & Future Flow | 10 | So sánh Rule vs LLM vs Agent; có AI step, HITL, fallback |
| G4. Decision Quality | 10 | Quyết định trung thực, có chứng cứ |

**Bảng chấm chọn bài** *(nhóm điền khi họp)*:

| Bài của | G1 (/20) | G2 (/20) | G3 (/10) | G4 (/10) | Tổng (/60) | Ghi chú |
|---|:-:|:-:|:-:|:-:|:-:|---|
| Bùi Ngọc Vũ | | | | | | |
| Nguyễn Thế Trường | | | | | | |
| Dương Tuấn Hưng | | | | | | |
| Tô Minh Thành | | | | | | |

---

## 3. Các file được merge vào `main`

| File | Lấy từ branch của | Lý do chọn |
|---|---|---|
| `01-problem-scan.md` | Bùi Ngọc Vũ | Đủ 5 bài toán theo các lens, có bảng chấm điểm chọn top 3 kèm lý do loại bài dễ làm bằng Rule, 3 Quick Cards có metric và phần stress-test (đóng vai CFO) dẫn tới thay đổi kiến trúc. Card #1 khớp với bài Deep-Dive được chọn. |
| `02-deep-dive-report.md` | Bùi Ngọc Vũ | Chi tiết nhất ở cả 4 gate: bảng workflow có input/output và thời gian từng bước (G1); 6-field với 4 nhóm metric có số và boundary rõ (G2); AI-Fit Matrix theo từng tác vụ, future flow có 5 kịch bản fallback và JSON output (G3); bảng rủi ro + GO có điều kiện, có tiêu chí quay về NOT YET (G4). Cùng bài toán Xanh SM với prompt prototype của lab. |
| `03-ai-log.md` | Bùi Ngọc Vũ | Ghi nhật ký từng lượt tương tác, có bảng lỗi/hallucination cụ thể, kết quả adversarial test chạy thật (3/3 PASS) và phân tích cả những lỗi script kiểm tra không bắt được. |
| `04-workflow-diagram.png` | Bùi Ngọc Vũ | Phải cùng bài toán với `02-deep-dive-report.md`. Thể hiện đủ 6 bước, 5 handoff, thời gian từng bước và 2 bottleneck. ⚠️ Cần sửa trước khi merge: một số nhãn bị cắt chữ ("HỆ THỐNG NỘI B…", "ĐỘI CỨU H…"). |

> **Phương án thay thế:** bài của **Dương Tuấn Hưng** (Vinhomes) cũng rất tốt: sơ đồ workflow rõ ràng, không lỗi hiển thị; metric chặt và quyết định GO trung thực (tự đánh dấu tiêu chí #3 chưa đạt). Nếu nhóm chọn bài này thì phải lấy **cả** `02-deep-dive-report.md` **và** `04-workflow-diagram.png` từ branch của Hưng để báo cáo và sơ đồ cùng một bài toán.

> 🛑 **Không** merge file `.py` vào `main`. File `starter-code/prompt_prototype.py` chỉ nằm trên branch cá nhân của từng thành viên.

**Lệnh merge (Trưởng nhóm thực hiện):**

```bash
git checkout main
git pull origin main
git checkout origin/2A202603463-BuiNgocVu -- 01-problem-scan.md 02-deep-dive-report.md 03-ai-log.md 04-workflow-diagram.png
git add TEAM.md
git commit -m "Chore: Select and merge best markdown reports into main"
git push origin main
```
