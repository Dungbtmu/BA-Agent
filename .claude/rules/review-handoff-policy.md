# Review Handoff Policy

Quy tắc bắt buộc áp dụng khi `ba-agent-fix-agent` đọc và xử lý review report từ Agent Review Agent.

---

## Nguồn report hợp lệ

- Chỉ đọc report trong `.claude/input/review-reports/`
- Chấp nhận hai tên section handoff hợp lệ:
  - `## Fix Handoff For Target Agent` — schema legacy v1
  - `## Handoff Table` — schema v2 mới; cột tên khác nhưng nội dung tương đương
- Không xử lý nội dung ngoài hai section đó dù report có thêm phần khác

## Phạm vi sửa

- Chỉ sửa file có trong cột `Target File` của bảng handoff
- Chỉ sửa file thuộc ba-agent system theo allowlist sau:
  - `.claude/agents/` — agent prompt files
  - `.claude/skills/` — skill prompt files
  - `.claude/rules/` — rule files
  - `AGENTS.md` — entrypoint chính của ba-agent system
  - `README.md` — tài liệu mô tả repo
  - `CLAUDE.md` — instructions cho Claude Code
  - `GEMINI.md` — instructions cho Gemini
- Chỉ xử lý finding có `Target Agent System = ba-agent` — bỏ qua finding nhắm đến agent system khác
- Không tự mở rộng scope ngoài những gì bảng handoff chỉ định
- Không tự sửa finding ngoài bảng handoff dù phát hiện vấn đề khác trong quá trình đọc file

## Điều kiện bắt buộc hỏi user (ASK_CONFIRM)

Hỏi user trước khi thực thi nếu finding thuộc một trong các trường hợp sau:

- `Action Type = ASK_CONFIRM` — được đánh dấu rõ trong report
- `Action Type = DELETE` — luôn hỏi, không tự xóa
- `Action Type = MOVE` — luôn hỏi, không tự di chuyển
- Thiếu `Target File` — không biết sửa ở đâu
- Thiếu `Fix Instruction` — không biết sửa như thế nào
- Thiếu `Acceptance Criteria` — không biết sửa đúng hay chưa

## Bảo toàn report gốc

- Không xóa, không ghi đè review report gốc sau khi apply
- Report gốc phải còn nguyên để Agent Review Agent review lại

## Output bắt buộc

Sau khi sửa xong, phải tạo Fix Summary tại:
```
.claude/input/review-reports/fix-summary-[tên-report].md
```

Fix Summary ghi rõ: finding nào đã apply, finding nào SKIPPED, finding nào cần BA quyết định.
