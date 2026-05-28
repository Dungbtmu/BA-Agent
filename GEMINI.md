# GEMINI.md

Tài liệu này cung cấp hướng dẫn cho Gemini (Antigravity) khi làm việc trong kho lưu trữ (repository) này.

Repo này là **BA Agent System** — hỗ trợ BA tạo tài liệu phân tích nghiệp vụ cho mọi dự án, mọi domain. Đầu ra chính là tài liệu **URD/SRS dev/test-ready**. Các tài liệu phụ trợ (Epic, User Story, AC, Wireframe, React prototype) chỉ được tạo khi BA yêu cầu cụ thể. Hệ thống không chứa mã nguồn sản phẩm (production code) cần build, test hay compile.

BA agent tập trung vào **nghiệp vụ và giao diện ở mức độ BA** (bao gồm việc tạo React prototype để lấy ý kiến phản hồi từ khách hàng/stakeholder), không xử lý thiết kế kiến trúc kỹ thuật chi tiết hay mã nguồn triển khai thực tế.

---

## 1. Bản đồ Phân vai (Subagents)

Dưới đây là các vai trò/subagent được định nghĩa trong thư mục [.claude/agents/](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/). Hãy đọc file định nghĩa tương ứng trước khi đóng vai subagent hoặc xử lý các tác vụ thuộc phạm vi tương ứng.

### Phase 1 — Làm rõ & Giải pháp

| Vai trò | File hướng dẫn | Kỹ năng liên quan (Skills) | Dùng khi nào |
|---|---|---|---|
| `ba-research-agent` | [ba-research-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-research-agent.md) | [domain-research.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/domain-research.md) | Khi nghiên cứu domain mới, chưa có kiến thức — nghiên cứu nghiệp vụ, actor, pain point, thuật ngữ để tạo Domain Brief. |
| `ba-clarification-agent` | [ba-clarification-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-clarification-agent.md) | [problem-framing.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/problem-framing.md), [requirement-clarification.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/requirement-clarification.md) | Khi có yêu cầu thô (mô tả sơ lược, ghi chú rời rạc) — làm rõ yêu cầu trước khi thiết kế. |
| `ba-solution-agent` | [ba-solution-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-solution-agent.md) | [user-persona-identification.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/user-persona-identification.md), [stakeholder-mapping.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/stakeholder-mapping.md), [assumption-risk-analysis.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/assumption-risk-analysis.md), [context-constraint-analysis.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/context-constraint-analysis.md) | Khi yêu cầu đã rõ — đề xuất giải pháp, thiết kế user flow, xác định edge cases và trade-offs. |
| `ba-backlog-agent` | [ba-backlog-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-backlog-agent.md) | [requirement-clarification.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/requirement-clarification.md), [user-persona-identification.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/user-persona-identification.md) | *(Optional)* Khi cần chia nhỏ giải pháp thành Epic, User Stories và AC rõ ràng, có khả năng kiểm thử. |

### Phase 2 — Thiết kế giao diện

| Vai trò | File hướng dẫn | Kỹ năng liên quan (Skills) | Dùng khi nào |
|---|---|---|---|
| `ba-wireframe-agent` | [ba-wireframe-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-wireframe-agent.md) | [wireframe-design-system.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/wireframe-design-system.md), [ui-feedback-triage.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/ui-feedback-triage.md) | Phác thảo màn hình, layout, UI flow dạng văn bản — áp dụng design system chuẩn chung hoặc per-project guideline. |
| `ui-react-agent` | [ui-react-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ui-react-agent.md) | [wireframe-design-system.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/wireframe-design-system.md), [react-ui-generation.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/react-ui-generation.md) | Phát triển React prototype nhanh để stakeholder xem trực quan và đưa phản hồi. |
| `ui-feedback-agent` | [ui-feedback-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ui-feedback-agent.md) | [ui-feedback-triage.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/ui-feedback-triage.md) | Nhận phản hồi thô về UI — phân loại, ánh xạ về đúng màn hình/thành phần và chỉ định sửa đổi chính xác. |

### Phase 3 — Tài liệu URD/SRS

| Vai trò | File hướng dẫn | Kỹ năng liên quan (Skills) | Dùng khi nào |
|---|---|---|---|
| `urd-srs-agent` | [urd-srs-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/urd-srs-agent.md) | [urd-srs-structure.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-srs-structure.md) cùng với các kỹ năng chi tiết theo phần ([workflow-diagram](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-workflow-diagram.md), [function-tree](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-function-tree.md), [permission-matrix](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-permission-matrix.md), [sequence-diagram](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-sequence-diagram.md), [use-case](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-use-case.md), [screen-spec](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-screen-spec.md)) | Viết tài liệu URD/SRS hoàn chỉnh, sẵn sàng cho đội ngũ Dev/Test — đây là sản phẩm đầu ra chính. |
| `ba-qa-agent` | [ba-qa-agent.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/agents/ba-qa-agent.md) | [urd-review-checklist.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/urd-review-checklist.md), [assumption-risk-analysis.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/assumption-risk-analysis.md), [requirement-clarification.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/requirement-clarification.md) | Đánh giá chất lượng tài liệu — phát hiện các lỗ hổng logic, trường hợp bị thiếu, xung đột, rủi ro. Tự động chạy sau khi `urd-srs-agent` hoàn thành. |

---

## 2. Quy trình Phối hợp 3 Phase

```
Phase 1 — Làm rõ & Giải pháp
  [0] ba-research-agent    (optional — chỉ khi gặp domain mới)
   ↓
  [1] ba-clarification-agent
   ↓
  [2] ba-solution-agent
   ↓
  [3] ba-backlog-agent     (optional — chỉ khi BA yêu cầu rõ về Epic/Story/AC)

Phase 2 — Thiết kế giao diện (vòng lặp đến khi thống nhất)
  [4] ba-wireframe-agent   ←→ hoạt động song song/phối hợp với
  [5] ui-react-agent
   ↓
  [6] ui-feedback-agent → phân loại (triage) → sửa đổi đúng vùng → lặp lại

Phase 3 — Tài liệu (vòng lặp đến khi hoàn thiện)
  [7] urd-srs-agent → soạn thảo URD/SRS
   ↓ (tự động kích hoạt)
  [8] ba-qa-agent → rà soát chất lượng → CRITICAL: bắt buộc sửa đổi đúng phần lỗi → lặp lại
                                        → MAJOR/MINOR: ghi nhận và báo cáo BA quyết định
                                        → Đạt yêu cầu: chốt tài liệu, lưu phiên bản chính thức
```

Có thể linh hoạt bắt đầu từ bất kỳ phase nào nếu đã có sẵn thông tin đầu vào phù hợp (ví dụ: đã có wireframe thì có thể bắt đầu luôn Phase 3).

---

## 3. Quy tắc & Chỉ dẫn dành riêng cho Gemini (Antigravity)

### Nạp Chỉ dẫn Nghiệp vụ & Kỹ năng
- Trước khi thực hiện bất kỳ tác vụ nào liên quan đến BA, Gemini **bắt buộc** phải đọc file [AGENTS.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/AGENTS.md) và các quy tắc chung trong thư mục [.claude/rules/](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/rules/).
- Đọc đúng file kỹ năng tương ứng trong thư mục [.claude/skills/](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/skills/) trước khi bắt đầu tạo hoặc chỉnh sửa bất kỳ phần nào của tài liệu đầu ra.

### Quy định về Ngôn ngữ & Định dạng
- **Ngôn ngữ:** Tất cả các câu trả lời, tài liệu phân tích nghiệp vụ, wireframe, và mô tả phải được viết bằng **tiếng Việt có dấu chuẩn chỉnh**.
- Cho phép sử dụng các thuật ngữ tiếng Anh phổ biến trong ngành (như *API, URD, SRS, Use Case, Actor, Wireframe, RBAC, Backlog, Epic*) hoặc khi tạo prompt cho Lovable. Không dịch gượng ép các thuật ngữ chuyên ngành này.
- **Đường dẫn & Liên kết:** Luôn tạo liên kết dạng markdown click được đến các file liên quan theo chuẩn: `[tên_file](file:///đường_dẫn_tuyệt_đối)`. Ví dụ: [[project-context.md](file:///Users/buidung/Documents/workspace/aeh/eduecosystem/ba-agent/.claude/rules/project-context.md)].

### Nguyên tắc Xử lý Thông tin & Tính xác thực
- **Không tự ý suy diễn kỹ thuật:** Không tự tạo ra các thông tin kỹ thuật chuyên sâu (như thiết kế Database Schema chi tiết, API Spec chi tiết, Kiến trúc hạ tầng mạng) khi đầu vào không yêu cầu hoặc không có sẵn thông tin bối cảnh. Tập trung hoàn toàn vào nghiệp vụ (business logic) và luồng đi của người dùng (user flow).
- **Quản lý thông tin thiếu:** Đối với các thông tin cực kỳ quan trọng nhưng chưa rõ (như danh sách Actor chính, phân quyền cốt lõi, quy trình cốt lõi), hãy nêu giả định rõ ràng và đánh dấu bằng thẻ `[Cần xác nhận: <nội dung>]` trong tài liệu, đồng thời tổng hợp lại danh sách câu hỏi để làm việc với người dùng.

### Quản lý File và Phiên bản (Version Control)
- **Tác vụ tạo mới (GENERATE):** Lưu sản phẩm đầu ra tại thư mục tương ứng theo mẫu: `.claude/output/[tên_dự_án]/[loại_tài_liệu]/[tên-file]-v[N].md`.
- **Tác vụ cập nhật (REFINE):** Chỉ chỉnh sửa đúng phạm vi được yêu cầu, giữ nguyên toàn bộ các nội dung không liên quan để bảo toàn tính toàn vẹn của tài liệu. Lưu phiên bản mới với số hiệu tăng tiến (ví dụ: `v1` -> `v2`), trừ khi người dùng yêu cầu ghi đè trực tiếp.

### Công cụ Khuyên dùng của Gemini (Antigravity)
- **Đọc file:** Sử dụng công cụ `view_file` để hiểu toàn bộ bối cảnh trước khi sửa đổi.
- **Chỉnh sửa file:**
  - Sử dụng `replace_file_content` cho các sửa đổi tập trung tại một vị trí liên tục.
  - Sử dụng `multi_replace_file_content` cho các sửa đổi rải rạc trên nhiều dòng không liền kề để tiết kiệm tài nguyên và tránh sai lệch.
- **Tạo giao diện mẫu (Mocks/Assets):** Sử dụng `generate_image` để phác thảo các UI mockups hoặc tạo asset cho sản phẩm/prototype khi có yêu cầu minh họa trực quan.
