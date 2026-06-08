# Agent Review Report — ba-agent System V2

**Ngày review:** 08/06/2026  
**Reviewer:** Agent Review Agent  
**Target Agent System:** ba-agent  
**Report ID:** BA-AGENT-SYSTEM-REVIEW-V2  
**Verdict:** NEEDS FIX  

---

## Findings

### CRITICAL-001 — Review/Fix bridge chưa hỗ trợ schema `Handoff Table` v2

**Severity:** CRITICAL  
**Location:** `.claude/rules/review-handoff-policy.md:9`, `.claude/agents/ba-agent-fix-agent.md:24`, `.claude/skills/shared/review-report-fix-handoff.md:70`

**Evidence**

- `review-handoff-policy.md` chỉ cho xử lý section `## Fix Handoff For Target Agent` và bỏ qua nội dung ngoài section đó.
- `ba-agent-fix-agent.md` cũng hard-code `Fix Handoff For Target Agent` và schema cột `Target Agent System`, `Target Section`, `Action Type`, `Fix Instruction`.
- `review-report-fix-handoff.md` hướng dẫn đọc đúng section legacy này, chưa mô tả alias section `## Handoff Table` và các cột `Repo/Module`, `Section/Anchor`, `Action`, `Required Change`.

**Impact**

Report v2 theo contract mới có thể bị `ba-agent-fix-agent` bỏ qua toàn bộ hoặc không map được cột, làm đứt pipeline Agent Review Agent -> ba-agent-fix-agent.

**Recommendation**

Cập nhật policy, fix agent và skill handoff để chấp nhận cả schema legacy và schema v2:

- Section hợp lệ: `## Fix Handoff For Target Agent` hoặc `## Handoff Table`.
- Alias cột: `Repo/Module` -> `Target Agent System`, `Section/Anchor` -> `Target Section`, `Action` -> `Action Type`, `Required Change` -> `Fix Instruction`.
- Giá trị `Repo/Module = ba-agent` được xử lý như `Target Agent System = ba-agent`.

**Acceptance Criteria**

- `ba-agent-fix-agent` parse được report có section `## Handoff Table` với 8 cột v2.
- Report legacy v1 vẫn parse được, không breaking change.
- Finding trong schema v2 không bị SKIPPED chỉ vì thiếu tên cột legacy.

---

### MAJOR-001 — Danh sách agent bắt buộc đọc trong entrypoint/context chưa bao phủ Phase 0 và Orchestrator

**Severity:** MAJOR  
**Location:** `AGENTS.md:29`, `AGENTS.md:67`, `.claude/rules/project-context.md:29`

**Evidence**

- `AGENTS.md:29-43` liệt kê các file agent cần đọc khi task tương ứng vai trò BA, nhưng thiếu `ba-brainstorm-agent.md` và `ba-orchestrator-agent.md`.
- Chính `AGENTS.md:67-83` lại định nghĩa cả `ba-brainstorm-agent` và `ba-orchestrator-agent` trong bảng subagent.
- `.claude/rules/project-context.md:29-43` liệt kê cây `.claude/agents/` nhưng thiếu `ba-orchestrator-agent.md`, trong khi file `.claude/agents/ba-orchestrator-agent.md` tồn tại và được dùng trong workflow.

**Impact**

Codex/subagent dispatch có thể nhận diện đúng intent brainstorm/orchestrate ở phần workflow nhưng không đọc prompt agent tương ứng ở bước load context ban đầu, làm giảm tính nhất quán khi chạy Phase 0 hoặc full pipeline/SYNC.

**Recommendation**

Đồng bộ danh sách agent ở entrypoint và project context:

- Thêm `.claude/agents/ba-brainstorm-agent.md` và `.claude/agents/ba-orchestrator-agent.md` vào danh sách agent cần đọc trong `AGENTS.md`.
- Thêm `ba-orchestrator-agent.md` vào cây thư mục trong `project-context.md`.

**Acceptance Criteria**

- Mọi agent tồn tại trong `.claude/agents/` và xuất hiện trong bảng subagent đều được phản ánh trong danh sách load agent hoặc cây context.
- Không còn lệch giữa `AGENTS.md` và `project-context.md` về danh sách agent chính.

---

### MAJOR-002 — Traceability Map chưa được đưa vào output schema dù là điều kiện bắt buộc của SYNC

**Severity:** MAJOR  
**Location:** `.claude/rules/output-schema.md:9`, `.claude/rules/output-schema.md:37`, `AGENTS.md:192`

**Evidence**

- `AGENTS.md:192-207` quy định Phase 3 tạo `[T] Traceability Map` và SYNC chỉ chạy khi có `.claude/output/[tên_dự_án]/traceability-map.md`.
- `.claude/rules/output-schema.md:9-15` chưa liệt kê Traceability Map trong artifact table.
- `.claude/rules/output-schema.md:37-42` traceability chain kết thúc ở Process Summary, chưa thể hiện Traceability Map là artifact sau Phase 3 và nền tảng cho SYNC.

**Impact**

Traceability Map là prerequisite vận hành của SYNC nhưng không được chuẩn hóa trong output schema. Agent có thể quên tạo, không kiểm tra version/history, hoặc xem đây là artifact phụ không bắt buộc.

**Recommendation**

Cập nhật `output-schema.md`:

- Thêm artifact `Traceability Map` với file `traceability-map.md`, trạng thái tự động sau Process Summary.
- Mở rộng traceability chain để thể hiện `URD/SRS -> Process Summary -> Traceability Map`.
- Ghi rõ Traceability Map là điều kiện bắt buộc trước SYNC.

**Acceptance Criteria**

- `output-schema.md` có dòng artifact Traceability Map.
- Traceability chain trong `output-schema.md` khớp với `AGENTS.md` và `agent-workflow.md`.
- SYNC prerequisite không chỉ nằm ở workflow mà còn nằm trong schema artifact.

---

### MAJOR-003 — Postcheck chưa có verdict rõ khi còn `[Cần xác nhận]`

**Severity:** MAJOR  
**Location:** `.claude/skills/urd/document-integrity-check.md:53`, `.claude/skills/urd/document-integrity-check.md:264`, `.claude/agents/ba-postcheck-agent.md:75`, `.claude/agents/urd-srs-agent.md:146`

**Evidence**

- `document-integrity-check.md:53-54` nói `[Cần xác nhận]` trong URD/SRS là WARN, nhưng trong tài liệu handoff cuối cho Dev/Test thì FAIL nếu là thông tin critical.
- `document-integrity-check.md:264-268` chỉ có verdict `READY FOR HANDOFF` hoặc `NEEDS FIX`, không hướng dẫn cách chuyển WARN thành verdict.
- `ba-postcheck-agent.md:75` nói để BA quyết định có chấp nhận handoff với các mục chưa xác nhận hay không, nhưng output format ở `ba-postcheck-agent.md:155-163` vẫn chỉ có verdict nhị phân.
- `urd-srs-agent.md:146` vẫn khẳng định Dev/Test đọc xong không cần hỏi thêm, trong khi cùng file cho phép ghi `[Cần xác nhận]`.

**Impact**

Cùng một tài liệu có thể vừa còn open confirmation vừa bị báo READY FOR HANDOFF, hoặc agent khác nhau tự diễn giải WARN thành FAIL/READY khác nhau. Điều này làm mơ hồ gate bàn giao cho Dev/Test.

**Recommendation**

Chốt rule nhất quán:

- `[Cần xác nhận]` liên quan Actor, phân quyền, trigger chính, quy trình chính hoặc scope chính -> `NEEDS FIX`.
- `[Cần xác nhận]` non-critical -> vẫn có thể `READY FOR HANDOFF` nhưng phải ghi rõ warning và đưa vào Process Summary/Handoff Note.
- Sửa wording `Dev/Test ready` trong `urd-srs-agent.md` thành điều kiện: chỉ "không cần hỏi thêm" khi không còn mục critical cần xác nhận.

**Acceptance Criteria**

- `document-integrity-check.md` có rule chuyển WARN/critical confirmation thành verdict rõ ràng.
- `ba-postcheck-agent.md` output format thể hiện warning còn lại nếu vẫn READY.
- `urd-srs-agent.md` không còn khẳng định tuyệt đối "không cần hỏi thêm" khi tài liệu còn `[Cần xác nhận]`.

---

### MAJOR-004 — SYNC conflict: `change-handler` tự tạo REQ mới khi không match Traceability Map

**Severity:** MAJOR  
**Location:** `.claude/skills/sync/change-handler.md:51`, `.claude/skills/sync/impact-analysis.md:56`

**Evidence**

- `change-handler.md:55-58` hướng dẫn: nếu không tìm được REQ trong Traceability Map thì tạo REQ ID mới.
- `impact-analysis.md:56` lại yêu cầu: nếu free text không mention REQ ID và không tìm được trong Traceability Map thì hỏi BA confirm trước khi tiếp tục.

**Impact**

Một thay đổi MODIFY nhưng match kém có thể bị tạo thành REQ mới, gây duplicate requirement, tính impact thiếu artifact cũ, và patch sai phạm vi.

**Recommendation**

Đồng bộ SYNC contract:

- Chỉ tạo REQ mới khi Change Type = ADD và BA đã xác nhận đây là requirement mới.
- Với MODIFY/REMOVE không match được REQ ID, dừng checkpoint hỏi BA chọn REQ tương ứng hoặc xác nhận chuyển thành ADD.
- Output Change Set phải ghi rõ `match_status`: MATCHED / NEW_CONFIRMED / AMBIGUOUS.

**Acceptance Criteria**

- `change-handler.md` không còn tự tạo REQ ID mới cho mọi trường hợp không match.
- `change-handler.md` và `impact-analysis.md` cùng dùng một rule khi Traceability Map không match.
- SYNC không chạy `artifact-patch` khi Change Set còn ambiguity về REQ ID.

---

### MAJOR-005 — Versioning backlog không nhất quán giữa agent tạo backlog và SYNC patch

**Severity:** MAJOR  
**Location:** `.claude/agents/ba-backlog-agent.md:72`, `.claude/skills/sync/artifact-patch.md:153`, `AGENTS.md:216`

**Evidence**

- `ba-backlog-agent.md:72-74` lưu backlog tại `.claude/output/[tên_dự_án]/backlog/backlog.md`.
- `artifact-patch.md:153` lại target backlog theo pattern `.claude/output/[tên_dự_án]/backlog/backlog-v[N].md`.
- `AGENTS.md:216-223` yêu cầu REFINE tăng version và không ghi đè file version cũ.

**Impact**

Nếu backlog được tạo bằng `ba-backlog-agent`, SYNC có thể không tìm thấy `backlog-v[N].md` để patch hoặc phải tự đoán từ `backlog.md`, làm sai versioning/audit trail.

**Recommendation**

Chuẩn hóa backlog output:

- Hoặc đổi `ba-backlog-agent` sang `backlog-v[N].md`.
- Hoặc cập nhật SYNC/artifact-patch để hỗ trợ `backlog.md` legacy và tạo version mới khi patch lần đầu.

**Acceptance Criteria**

- Backlog generate/refine/SYNC dùng cùng một versioning contract.
- `artifact-patch` không target file name mà `ba-backlog-agent` không tạo ra.
- Quy tắc không ghi đè version cũ áp dụng rõ cho backlog nếu backlog tồn tại.

---

### MINOR-001 — Root docs phụ trợ chưa nhắc đủ `review-handoff-policy`

**Severity:** MINOR  
**Location:** `CLAUDE.md:117`, `GEMINI.md:58`, `AGENTS.md:27`

**Evidence**

- `AGENTS.md:27` đã đưa `.claude/rules/review-handoff-policy.md` vào rule bắt buộc.
- `ba-agent-fix-agent.md:10-12` yêu cầu đọc cả `review-handoff-policy.md` và `review-report-fix-handoff.md`.
- `CLAUDE.md:117-126` liệt kê rules chi tiết nhưng không nhắc `review-handoff-policy.md`.
- `GEMINI.md:58` mô tả `ba-agent-fix-agent` với skill handoff nhưng không nhắc policy bắt buộc.

**Impact**

Người vận hành qua Claude/Gemini docs có thể load thiếu policy allowlist/ASK_CONFIRM khi apply review report, dù `AGENTS.md` đã đúng.

**Recommendation**

Đồng bộ docs phụ trợ:

- Thêm `review-handoff-policy.md` vào danh sách rules trong `CLAUDE.md`.
- Thêm policy này vào mô tả `ba-agent-fix-agent` trong `GEMINI.md`.

**Acceptance Criteria**

- CLAUDE/GEMINI đều nhắc đủ policy và skill cho review/fix bridge.
- Root docs không tạo cảm giác chỉ cần đọc skill mà bỏ qua policy.

---

## Summary

| Severity | Số lượng |
|---|---:|
| CRITICAL | 1 |
| MAJOR | 5 |
| MINOR | 1 |

**Không còn CRITICAL từ review v1 chưa fix.** CRITICAL mới của v2 là contract drift giữa schema handoff mới (`Handoff Table`) và parser hiện tại của `ba-agent-fix-agent`.

---

## Handoff Table

| Finding ID | Severity | Repo/Module | Target File | Section/Anchor | Action | Required Change | Acceptance Criteria |
|---|---|---|---|---|---|---|---|
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/rules/review-handoff-policy.md` | `Nguồn report hợp lệ` | UPDATE | Cho phép xử lý cả section `## Fix Handoff For Target Agent` và `## Handoff Table`; ghi rõ `Handoff Table` là schema v2 hợp lệ. | Policy không bỏ qua report chỉ vì dùng section `Handoff Table`. |
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/agents/ba-agent-fix-agent.md` | `Bước 1 — Parse report` | UPDATE | Bổ sung alias mapping: `Repo/Module` -> `Target Agent System`, `Section/Anchor` -> `Target Section`, `Action` -> `Action Type`, `Required Change` -> `Fix Instruction`; chấp nhận `Repo/Module = ba-agent`. | Agent parse được cả schema legacy và schema v2, không SKIPPED finding vì tên cột v2. |
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/skills/shared/review-report-fix-handoff.md` | `Schema finding hợp lệ` | UPDATE | Mô tả schema v2 `Handoff Table` với 8 cột và rule map sang schema nội bộ hiện tại. | Skill handoff document hóa đầy đủ schema v2 và backward compatibility với v1. |
| MAJOR-001 | MAJOR | ba-agent | `AGENTS.md` | `Khi tác vụ tương ứng với một vai trò BA chuyên biệt` | UPDATE | Thêm `.claude/agents/ba-brainstorm-agent.md` và `.claude/agents/ba-orchestrator-agent.md` vào danh sách agent cần đọc. | Danh sách agent cần đọc khớp với bảng subagent và không thiếu Phase 0/Orchestrator. |
| MAJOR-001 | MAJOR | ba-agent | `.claude/rules/project-context.md` | `Cấu trúc thư mục > agents` | UPDATE | Thêm `ba-orchestrator-agent.md` vào cây `.claude/agents/`. | Cây thư mục phản ánh đầy đủ các agent đang tồn tại và được workflow sử dụng. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/rules/output-schema.md` | `Artifact và ID format` | UPDATE | Thêm `Traceability Map` với file `traceability-map.md`, trạng thái tự động sau Process Summary và là prerequisite của SYNC. | Artifact table có Traceability Map và mô tả đúng vai trò trong SYNC. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/rules/output-schema.md` | `Traceability chain` | UPDATE | Cập nhật chain thành `... -> URD/SRS -> Process Summary -> Traceability Map`; ghi rõ map là nền tảng cho SYNC. | Chain khớp với AGENTS.md và agent-workflow.md. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/skills/urd/document-integrity-check.md` | `Kiểm tra bổ sung — [Cần xác nhận]` | UPDATE | Bổ sung rule phân loại critical/non-critical confirmation và cách chuyển thành verdict. | Critical confirmation còn mở dẫn đến NEEDS FIX; non-critical confirmation có thể READY nhưng phải warning rõ. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/agents/ba-postcheck-agent.md` | `Output format / Phán quyết` | UPDATE | Cho phép output READY kèm warning hoặc NEEDS FIX theo rule critical confirmation; không để BA tự đoán từ WARN. | Postcheck report có verdict rõ khi còn `[Cần xác nhận]`. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/agents/urd-srs-agent.md` | `Nguyên tắc bất biến` | UPDATE | Sửa câu "không cần hỏi thêm" thành điều kiện chỉ đúng khi không còn critical `[Cần xác nhận]`; nếu còn thì phải đưa vào danh sách cần xác nhận/handoff. | URD/SRS agent không tự mâu thuẫn giữa allowed confirmation và Dev/Test ready. |
| MAJOR-004 | MAJOR | ba-agent | `.claude/skills/sync/change-handler.md` | `Bước 2 — Tìm REQ ID trong Traceability Map` | UPDATE | Không tự tạo REQ mới khi không match cho MODIFY/REMOVE; chỉ tạo khi ADD được BA xác nhận; thêm `match_status`. | Change Set không tạo duplicate REQ do match kém. |
| MAJOR-004 | MAJOR | ba-agent | `.claude/skills/sync/impact-analysis.md` | `Bước 1 — Parse thay đổi` | UPDATE | Đồng bộ với `change-handler`: nếu Change Set còn `AMBIGUOUS` thì dừng hỏi BA, không phân tích impact. | SYNC không patch artifact khi chưa xác định đúng REQ ID. |
| MAJOR-005 | MAJOR | ba-agent | `.claude/agents/ba-backlog-agent.md` | `Lưu file` | UPDATE | Chuẩn hóa output backlog sang `backlog-v[N].md` hoặc ghi rõ legacy `backlog.md` sẽ được version hóa ở lần refine/SYNC đầu tiên. | Backlog generate/refine/SYNC có cùng naming contract. |
| MAJOR-005 | MAJOR | ba-agent | `.claude/skills/sync/artifact-patch.md` | `Patch Backlog / Story / AC` | UPDATE | Đồng bộ target backlog với output của `ba-backlog-agent`; nếu hỗ trợ legacy thì quy định cách tìm `backlog.md` và tạo `backlog-v[N].md`. | Artifact patch không target file mà agent generate không tạo. |
| MINOR-001 | MINOR | ba-agent | `CLAUDE.md` | `Rules chi tiết` | UPDATE | Thêm `.claude/rules/review-handoff-policy.md` vào danh sách rule chi tiết. | Claude docs nhắc đủ policy của review/fix bridge. |
| MINOR-001 | MINOR | ba-agent | `GEMINI.md` | `ba-agent-fix-agent` row | UPDATE | Bổ sung `review-handoff-policy.md` là policy bắt buộc cùng với `review-report-fix-handoff.md`. | Gemini docs không bỏ sót policy allowlist/ASK_CONFIRM. |
