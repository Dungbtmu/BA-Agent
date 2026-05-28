# Agent Workflow — Quy trình phối hợp các subagent BA

**Output chính của hệ thống là URD/SRS.** Epic, User Story, AC là artifact phụ trợ — chỉ tạo khi BA yêu cầu rõ.

## Phase 1 — Làm rõ & Giải pháp

```
Input (mọi dạng: mô tả thô, ghi chú, tài liệu chưa đầy đủ...)
   ↓
[0] ba-research-agent       → research domain mới, tạo Domain Brief  *(optional — chỉ khi BA chưa biết domain)*
   ↓
[1] ba-clarification-agent  → làm rõ yêu cầu, đặt câu hỏi clarify
   ↓ (nếu còn câu hỏi CRITICAL → dừng hỏi BA/PO trước)
[2] ba-solution-agent       → đề xuất solution + user flow
   ↓
[3] ba-backlog-agent        → Epic + User Stories + AC  *(optional — chỉ khi BA yêu cầu)*
```

## Phase 2 — Thiết kế giao diện (vòng lặp đến khi chốt)

```
[4] ba-wireframe-agent      → phác thảo màn hình + UI flow (text-based)
    ↕ song song
[5] ui-react-agent          → gen React prototype để xem trực quan
   ↓
[6] ui-feedback-agent       → nhận feedback thô → phân loại → chỉ định sửa đúng chỗ
   ↓ (lặp lại [4][5][6] cho đến khi stakeholder chốt giao diện)
```

## Phase 3 — Tài liệu (vòng lặp đến khi chốt)

```
[7] urd-srs-agent           → viết tài liệu URD/SRS Dev/Test-ready
   ↓ (tự động)
[8] ba-qa-agent             → review toàn bộ, phân loại CRITICAL/MAJOR/MINOR
   ↓
   ├─ Có CRITICAL → xác định đúng section bị lỗi → quay lại [7] sửa đúng chỗ đó
   ├─ Chỉ MAJOR/MINOR → báo cáo cho BA quyết định có sửa không
   └─ Không có issue → chốt tài liệu, lưu file version cuối
   ↓ (lặp lại [7][8] cho đến khi không còn CRITICAL)
```

**Quy tắc vòng lặp Phase 3:**
- Sau khi `urd-srs-agent` viết xong → **tự động** gọi `ba-qa-agent` review, không cần BA nhắc
- Mỗi lần sửa chỉ sửa đúng section có CRITICAL — không viết lại toàn bộ tài liệu
- MAJOR/MINOR: liệt kê trong báo cáo, BA quyết định sửa ngay hay để version sau
- Chốt khi không còn CRITICAL issue — lưu file với version tăng (`urd-srs-v[N+1].md`)

## Nhận dạng intent — tự dispatch agent

Khi BA không mention agent cụ thể, nhận dạng intent từ input và tự gọi agent phù hợp:

| Dấu hiệu trong input | Agent phù hợp |
|---|---|
| "tôi chưa biết gì về domain này", "research", "tìm hiểu lĩnh vực", "dự án mới chưa có kiến thức" | `ba-research-agent` |
| Mô tả dự án mới, yêu cầu chưa rõ, "tôi có dự án...", "tôi nhận được yêu cầu..." | `ba-clarification-agent` |
| Đã rõ yêu cầu, "đề xuất giải pháp", "flow nên như thế nào", "thiết kế hệ thống" | `ba-solution-agent` |
| "viết wireframe", "phác thảo màn hình", "UI flow", "layout" | `ba-wireframe-agent` |
| "code React", "prototype", "xem trực quan", "chạy thử" | `ui-react-agent` |
| Feedback từ stakeholder, "họ comment...", "sửa UI", "góp ý về giao diện" | `ui-feedback-agent` |
| "viết URD", "viết SRS", "tài liệu đặc tả", "đặc tả yêu cầu" | `urd-srs-agent` |
| "review lại", "check quality", "còn thiếu gì", "kiểm tra tài liệu" | `ba-qa-agent` |
| "viết Epic", "viết User Story", "chia backlog", "AC" | `ba-backlog-agent` |

Nếu input không khớp rõ → hỏi BA muốn làm gì trước khi dispatch.

## Nguyên tắc

- Chấp nhận mọi dạng input — không yêu cầu PRD chuẩn
- Output của bước trước = input của bước sau
- `ba-backlog-agent` là optional — chỉ dùng khi BA cần viết Epic/User Story/AC; có thể bỏ qua và đi thẳng sang Phase 2
- Có thể skip bước nếu BA yêu cầu rõ (ví dụ: đã có wireframe rồi, bắt đầu từ Phase 3)
- Nếu input thiếu thông tin → nêu assumption rõ ràng, tiếp tục dựa trên những gì đã có
- Phase 2 và Phase 3 là vòng lặp — không có số lần cố định, chốt khi stakeholder confirm
- Feedback từ Phase 2 chỉ ảnh hưởng wireframe/UI — không tự thay đổi Epic/Story trừ khi BA yêu cầu
