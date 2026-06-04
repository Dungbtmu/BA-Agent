# BA-Agent

Hệ thống agent hỗ trợ BA tạo tài liệu phân tích nghiệp vụ — từ nghiên cứu domain, làm rõ yêu cầu, thiết kế giao diện đến viết URD/SRS dev/test-ready.

## Entrypoint

- **`AGENTS.md`** — entrypoint chính, mô tả toàn bộ agent, skill, rule và workflow 3 phase. Đọc file này trước khi làm bất kỳ tác vụ BA nào.
- **`.claude/rules/`** — rule bắt buộc, load mọi session.
- **`.claude/agents/`** — prompt định nghĩa từng vai trò BA chuyên biệt.
- **`.claude/skills/`** — skill hỗ trợ theo từng tác vụ cụ thể.
- **`.claude/output/[tên_dự_án]/`** — tài liệu đầu ra theo dự án.

## Review/Fix Bridge

Hệ thống có pipeline nhận và apply review report từ Agent Review Agent:

1. Agent Review Agent export report vào `.claude/input/review-reports/[tên-report].md`
2. Dùng `ba-agent-fix-agent` để đọc report và apply fix — agent sẽ hiển thị Pre-flight summary trước khi sửa
3. Kết quả lưu tại `.claude/input/review-reports/fix-summary-[tên-report].md`
4. Copy Fix Summary sang Agent Review Agent để xác nhận các fix đã đúng

Cách kích hoạt `ba-agent-fix-agent`: mô tả "apply review report" hoặc "sửa theo review report" trong prompt — agent tự nhận dạng intent và load đúng agent.
