# BA Workflow — Quy trình tạo tài liệu

## Đầu vào

Chấp nhận mọi dạng input — không yêu cầu tài liệu chuẩn:

- Mô tả miệng / ý tưởng thô
- Ghi chú rời, bullet points
- Tài liệu sơ bộ chưa đầy đủ
- PRD (nếu có)

Khi input thiếu thông tin: nêu assumption rõ ràng, tiếp tục dựa trên những gì đã có. KHÔNG chặn workflow vì thiếu tài liệu.

BA agent chỉ tập trung vào nghiệp vụ. KHÔNG bịa đặt thông tin kỹ thuật (Architecture, API, Data, Event...).

## Chế độ GENERATE — Tạo mới từ đầu

**Output chính là URD/SRS.** Các artifact trung gian (Epic, User Story, AC) chỉ dùng khi BA thấy cần thiết để làm rõ yêu cầu trước khi viết tài liệu — không phải bước bắt buộc.

Workflow chia 3 phase, có thể bắt đầu từ bất kỳ phase nào nếu BA chỉ định rõ:

**Phase 1 — Làm rõ & Giải pháp**
```
Làm rõ yêu cầu → Đề xuất solution → [Epic + User Stories + AC — chỉ khi BA yêu cầu]
```
- Mặc định: làm rõ xong → đề xuất solution/user flow → đi thẳng sang Phase 2
- Epic/Story/AC chỉ tạo khi BA yêu cầu rõ — đây là công cụ phụ trợ, không phải đầu ra

**Phase 2 — Giao diện (vòng lặp)**
```
Wireframe (text) → React prototype → Nhận feedback → Triage → Sửa đúng chỗ → (lặp lại)
```
- Wireframe và React prototype chạy song song khi cần xem trực quan
- Mỗi vòng feedback: triage trước, sửa sau — không sửa khi chưa rõ
- Chỉ sửa đúng phần được yêu cầu — không tái tạo toàn bộ
- Kết thúc khi BA/stakeholder chốt giao diện

**Phase 3 — Tài liệu (vòng lặp)**
```
Viết URD/SRS → Review → Sửa → (lặp lại đến khi chốt)
```
- Bắt đầu sau khi giao diện đã chốt ở Phase 2
- Mỗi vòng review: phân loại CRITICAL/MAJOR/MINOR, có khuyến nghị cụ thể
- Kết thúc khi không còn CRITICAL issue

Đầu ra lưu tại: `output/v1/` (lần đầu), `output/v[N+1]/` (các lần sau)

## Chế độ REFINE — Chỉnh sửa có mục tiêu

- Chỉ sửa **đúng phần được yêu cầu**
- **KHÔNG tái tạo toàn bộ tài liệu** — đây là lỗi nghiêm trọng nhất
- Giữ nguyên toàn bộ nội dung không liên quan đến yêu cầu
- Áp dụng cho cả wireframe, React component, và URD/SRS
- Đầu ra lưu tại: `output/v[N+1]/`

## Chế độ REVIEW — Kiểm tra chất lượng

- Phân loại vấn đề theo 3 mức: CRITICAL / MAJOR / MINOR
- Mỗi vấn đề phải kèm khuyến nghị cụ thể, có thể hành động được
- CRITICAL → phải sửa trước khi tiếp tục phase tiếp theo
- Sau review: nếu có CRITICAL → quay lại đúng bước có vấn đề, không làm lại toàn bộ
