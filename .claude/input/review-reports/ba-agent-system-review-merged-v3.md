# Agent Review Report — ba-agent System Merged V3

**Ngày merge:** 08/06/2026  
**Nguồn:** `ba-agent-system-review-v2.md` + self-review table từ ba-agent  
**Target Agent System:** ba-agent  
**Report ID:** BA-AGENT-SYSTEM-REVIEW-MERGED-V3  
**Verdict:** NEEDS FIX  

---

## Findings

### CRITICAL-001 — Review/Fix bridge chưa hỗ trợ schema `Handoff Table` v2

**Severity:** CRITICAL  
**Location:** `.claude/rules/review-handoff-policy.md:9`, `.claude/agents/ba-agent-fix-agent.md:24`, `.claude/skills/shared/review-report-fix-handoff.md:70`

**Evidence**

- `review-handoff-policy.md` chỉ cho xử lý section `## Fix Handoff For Target Agent`.
- `ba-agent-fix-agent.md` hard-code schema legacy với các cột `Target Agent System`, `Target Section`, `Action Type`, `Fix Instruction`.
- `review-report-fix-handoff.md` chưa mô tả alias section `## Handoff Table` và các cột v2: `Repo/Module`, `Section/Anchor`, `Action`, `Required Change`.

**Impact**

Report theo schema mới có thể bị `ba-agent-fix-agent` bỏ qua hoặc parse thiếu cột, làm đứt pipeline Agent Review Agent -> ba-agent-fix-agent.

**Recommendation**

Cập nhật policy, fix agent và skill handoff để chấp nhận cả schema legacy và schema v2.

**Acceptance Criteria**

- `ba-agent-fix-agent` parse được cả `## Fix Handoff For Target Agent` và `## Handoff Table`.
- Alias cột v2 được map về schema nội bộ hiện tại.
- Report legacy v1 vẫn parse được.

---

### CRITICAL-002 — Broken path đến `ba-conventions.md` trong skill clarification/shared

**Severity:** CRITICAL  
**Location:** `.claude/skills/clarification/requirement-clarification.md:150`, `.claude/skills/shared/resolve-oqs.md:177`

**Evidence**

- `requirement-clarification.md` dùng reference `@../rules/ba-conventions.md`, nhưng từ thư mục `.claude/skills/clarification/` path này trỏ tới `.claude/skills/rules/ba-conventions.md`, không tồn tại.
- `resolve-oqs.md` dùng reference `@../rules/ba-conventions.md`, nhưng từ thư mục `.claude/skills/shared/` path này cũng trỏ sai.
- Self-review table đã flag lỗi này là CR-01.

**Impact**

Agent/skill có thể không load được rule nền IT-BA framing, no-re-ask, assumption, approval gate và OQ format. Đây là rule chung bắt buộc cho output BA.

**Recommendation**

Chuẩn hóa reference về `.claude/rules/ba-conventions.md` hoặc path tương đối đúng từ từng thư mục skill.

**Acceptance Criteria**

- Hai skill trỏ đúng đến `.claude/rules/ba-conventions.md`.
- Không còn reference `@../rules/ba-conventions.md` sai cấp thư mục trong `.claude/skills/clarification/` và `.claude/skills/shared/`.

---

### CRITICAL-003 — Danh sách agent bắt buộc đọc trong entrypoint/context chưa bao phủ Phase 0 và Orchestrator

**Severity:** CRITICAL  
**Location:** `AGENTS.md:29`, `AGENTS.md:67`, `.claude/rules/project-context.md:29`

**Evidence**

- `AGENTS.md:29-43` liệt kê các file agent cần đọc khi task tương ứng vai trò BA, nhưng thiếu `ba-brainstorm-agent.md` và `ba-orchestrator-agent.md`.
- `AGENTS.md:67-83` lại định nghĩa cả `ba-brainstorm-agent` và `ba-orchestrator-agent` trong bảng subagent.
- `.claude/rules/project-context.md:29-43` liệt kê cây `.claude/agents/` nhưng thiếu `ba-orchestrator-agent.md`.
- Self-review table flag cùng nhóm lỗi ở CR-02 và CR-03.

**Impact**

Codex/subagent dispatch có thể nhận diện đúng intent brainstorm/orchestrate nhưng không đọc prompt agent tương ứng ở bước load context ban đầu, ảnh hưởng Phase 0, full pipeline và SYNC.

**Recommendation**

Đồng bộ danh sách agent trong `AGENTS.md` và `project-context.md`.

**Acceptance Criteria**

- `AGENTS.md` thêm `.claude/agents/ba-brainstorm-agent.md` và `.claude/agents/ba-orchestrator-agent.md` vào danh sách agent cần đọc.
- `project-context.md` thêm `ba-orchestrator-agent.md` vào cây `.claude/agents/`.

---

### MAJOR-001 — CLAUDE/GEMINI mô tả Phase 2 chạy song song, mâu thuẫn với `agent-workflow.md`

**Severity:** MAJOR  
**Location:** `CLAUDE.md:83`, `GEMINI.md:81`, `.claude/rules/agent-workflow.md:39`

**Evidence**

- `CLAUDE.md:83-85` mô tả `ba-wireframe-agent` song song với `ui-react-agent`.
- `GEMINI.md:81-83` mô tả `ba-wireframe-agent` hoạt động song song/phối hợp với `ui-react-agent`.
- `agent-workflow.md:39-48` quy định `ui-react-agent` phụ thuộc vào wireframe/solution đủ rõ và không chạy song song với bước tạo wireframe ngay từ đầu.
- Self-review table flag lỗi này là MA-01.

**Impact**

Các môi trường vận hành khác nhau có thể dispatch sai thứ tự, tạo prototype trước khi wireframe đủ rõ, làm lệch Phase 2.

**Recommendation**

Sửa `CLAUDE.md` và `GEMINI.md` để Phase 2 là tuần tự: Wireframe -> React Prototype -> Feedback Triage. Chỉ cho phép song song ở các bước review độc lập.

**Acceptance Criteria**

- Root docs không còn mô tả `[4]` và `[5]` chạy song song trong luồng tạo mới.
- Mô tả Phase 2 khớp với `agent-workflow.md`.

---

### MAJOR-002 — Traceability Map chưa được đưa vào output schema dù là điều kiện bắt buộc của SYNC

**Severity:** MAJOR  
**Location:** `.claude/rules/output-schema.md:9`, `.claude/rules/output-schema.md:37`, `AGENTS.md:192`

**Evidence**

- `AGENTS.md:192-207` quy định Phase 3 tạo Traceability Map và SYNC chỉ chạy khi có `.claude/output/[tên_dự_án]/traceability-map.md`.
- `.claude/rules/output-schema.md:9-15` chưa liệt kê Traceability Map trong artifact table.
- `.claude/rules/output-schema.md:37-42` traceability chain kết thúc ở Process Summary.

**Impact**

Traceability Map là prerequisite vận hành của SYNC nhưng không được chuẩn hóa trong output schema.

**Recommendation**

Thêm Traceability Map vào `output-schema.md`, mở rộng traceability chain và ghi rõ đây là điều kiện bắt buộc trước SYNC.

**Acceptance Criteria**

- `output-schema.md` có dòng artifact Traceability Map.
- Traceability chain khớp `AGENTS.md` và `agent-workflow.md`.

---

### MAJOR-003 — Postcheck chưa có verdict rõ khi còn `[Cần xác nhận]`

**Severity:** MAJOR  
**Location:** `.claude/skills/urd/document-integrity-check.md:53`, `.claude/skills/urd/document-integrity-check.md:264`, `.claude/agents/ba-postcheck-agent.md:75`, `.claude/agents/urd-srs-agent.md:146`

**Evidence**

- `document-integrity-check.md` xem `[Cần xác nhận]` là WARN, nhưng chưa có rule chuyển WARN thành verdict rõ.
- `ba-postcheck-agent.md` để BA quyết định có handoff với mục chưa xác nhận hay không, trong khi output format vẫn thiên về verdict nhị phân.
- `urd-srs-agent.md` khẳng định Dev/Test đọc xong không cần hỏi thêm, dù hệ thống cho phép `[Cần xác nhận]`.

**Impact**

Một tài liệu có thể còn confirmation critical nhưng vẫn bị hiểu là READY FOR HANDOFF.

**Recommendation**

Phân loại `[Cần xác nhận]` critical/non-critical và quy định verdict tương ứng.

**Acceptance Criteria**

- Critical confirmation còn mở dẫn tới `NEEDS FIX`.
- Non-critical confirmation có thể `READY FOR HANDOFF` nhưng phải warning rõ.
- `urd-srs-agent.md` không còn khẳng định tuyệt đối khi còn critical confirmation.

---

### MAJOR-004 — SYNC conflict: `change-handler` tự tạo REQ mới khi không match Traceability Map

**Severity:** MAJOR  
**Location:** `.claude/skills/sync/change-handler.md:51`, `.claude/skills/sync/impact-analysis.md:56`

**Evidence**

- `change-handler.md` hướng dẫn nếu không tìm được REQ trong Traceability Map thì tạo REQ ID mới.
- `impact-analysis.md` lại yêu cầu nếu free text không mention REQ ID và không tìm được trong Traceability Map thì hỏi BA confirm trước.

**Impact**

Một thay đổi MODIFY nhưng match kém có thể bị tạo thành REQ mới, gây duplicate requirement và patch sai phạm vi.

**Recommendation**

Chỉ tạo REQ mới khi Change Type = ADD và BA đã xác nhận đây là requirement mới. Với MODIFY/REMOVE không match được, dừng checkpoint.

**Acceptance Criteria**

- `change-handler.md` không tự tạo REQ mới cho MODIFY/REMOVE không match.
- Change Set có `match_status`: MATCHED / NEW_CONFIRMED / AMBIGUOUS.
- SYNC không chạy patch khi còn ambiguity về REQ ID.

---

### MAJOR-005 — Versioning backlog không nhất quán giữa agent tạo backlog và SYNC patch

**Severity:** MAJOR  
**Location:** `.claude/agents/ba-backlog-agent.md:72`, `.claude/skills/sync/artifact-patch.md:153`, `AGENTS.md:216`

**Evidence**

- `ba-backlog-agent.md` lưu backlog tại `.claude/output/[tên_dự_án]/backlog/backlog.md`.
- `artifact-patch.md` target backlog theo pattern `.claude/output/[tên_dự_án]/backlog/backlog-v[N].md`.
- `AGENTS.md:216-223` yêu cầu REFINE tăng version và không ghi đè file version cũ.

**Impact**

SYNC có thể không tìm thấy file backlog để patch hoặc phải tự đoán từ `backlog.md`.

**Recommendation**

Chuẩn hóa backlog output sang `backlog-v[N].md` hoặc document rõ legacy `backlog.md` sẽ được version hóa ở lần refine/SYNC đầu tiên.

**Acceptance Criteria**

- Backlog generate/refine/SYNC dùng cùng một versioning contract.
- `artifact-patch` không target file mà `ba-backlog-agent` không tạo ra.

---

### MAJOR-006 — Cây output trong project context thiếu `brainstorm/` và `backlog/`

**Severity:** MAJOR  
**Location:** `.claude/rules/project-context.md:93`, `.claude/agents/ba-brainstorm-agent.md:40`, `.claude/agents/ba-backlog-agent.md:74`

**Evidence**

- `project-context.md:93-101` mô tả output project gồm `research/`, `wireframe/`, `urd/`, `solution/`, `process-summary.md`.
- `ba-brainstorm-agent.md` lưu output vào `.claude/output/[tên_dự_án]/brainstorm/{idea-slug}.md`.
- `ba-backlog-agent.md` lưu output vào `.claude/output/[tên_dự_án]/backlog/backlog.md`.
- Self-review table flag lỗi này là MA-02.

**Impact**

Project context không phản ánh đủ artifact thật, khiến người vận hành hoặc agent khác bỏ sót output Phase 0/backlog khi audit hoặc sync.

**Recommendation**

Thêm `brainstorm/` và `backlog/` vào cây output trong `project-context.md`.

**Acceptance Criteria**

- Cây output project có `brainstorm/` và `backlog/`.
- Mô tả output khớp các agent tạo artifact tương ứng.

---

### MAJOR-007 — `.claude/input/change-requests/` được dùng trong SYNC nhưng chưa được document trong project context

**Severity:** MAJOR  
**Location:** `.claude/agents/ba-orchestrator-agent.md:236`, `.claude/rules/project-context.md:90`

**Evidence**

- `ba-orchestrator-agent.md` dùng trigger file PO mới tại `.claude/input/change-requests/`.
- `project-context.md:90-92` chỉ document `.claude/input/review-reports/` trong cây input, chưa có `change-requests/`.
- Self-review table flag lỗi này là MA-03.

**Impact**

SYNC input folder không được chuẩn hóa, làm agent/user khó biết đặt change request ở đâu.

**Recommendation**

Bổ sung `.claude/input/change-requests/` vào cây input và mô tả đây là inbox cho PO update/change request phục vụ SYNC.

**Acceptance Criteria**

- `project-context.md` có `change-requests/`.
- Mô tả SYNC input khớp `ba-orchestrator-agent.md`.

---

### MAJOR-008 — `diagram` skill chưa được wired đầy đủ vào root docs

**Severity:** MAJOR  
**Location:** `AGENTS.md:125`, `CLAUDE.md:54`, `.claude/rules/agent-workflow.md:100`

**Evidence**

- `agent-workflow.md:100` có intent mapping cho `diagram` skill.
- `AGENTS.md:125-158` liệt kê skill bắt buộc theo tác vụ nhưng chưa có `.claude/skills/diagram/SKILL.md`.
- `CLAUDE.md` không mô tả diagram skill trong phần công cụ/hệ thống.
- Self-review table flag lỗi này là MA-04.

**Impact**

Người vận hành có thể nhận diện intent vẽ diagram nhưng không load skill Excalidraw đúng cách, bỏ qua workflow render-verify bắt buộc.

**Recommendation**

Thêm `diagram` skill vào danh sách skill/task mapping trong `AGENTS.md` và root docs phụ trợ.

**Acceptance Criteria**

- `AGENTS.md` nêu rõ khi vẽ diagram phải đọc `.claude/skills/diagram/SKILL.md`.
- `CLAUDE.md` nhắc diagram skill hoặc trỏ về `agent-workflow.md` đủ rõ cho intent diagram.

---

### MINOR-001 — Root docs phụ trợ chưa nhắc đủ `review-handoff-policy`

**Severity:** MINOR  
**Location:** `CLAUDE.md:117`, `GEMINI.md:58`, `AGENTS.md:27`

**Evidence**

- `AGENTS.md:27` đã đưa `.claude/rules/review-handoff-policy.md` vào rule bắt buộc.
- `CLAUDE.md:117-126` liệt kê rules chi tiết nhưng không nhắc `review-handoff-policy.md`.
- `GEMINI.md:58` mô tả `ba-agent-fix-agent` với skill handoff nhưng không nhắc policy bắt buộc.

**Impact**

Người vận hành qua Claude/Gemini docs có thể load thiếu policy allowlist/ASK_CONFIRM khi apply review report.

**Recommendation**

Đồng bộ root docs phụ trợ để nhắc đủ policy và skill cho review/fix bridge.

**Acceptance Criteria**

- `CLAUDE.md` thêm `.claude/rules/review-handoff-policy.md`.
- `GEMINI.md` nhắc `review-handoff-policy.md` cùng `review-report-fix-handoff.md`.

---

### MINOR-002 — Naming/comment style chưa nhất quán ở một số skill/agent

**Severity:** MINOR  
**Location:** `.claude/skills/clarification/brainstorm.md`, `.claude/agents/ba-clarification-agent.md`

**Evidence**

- Self-review table gom các lỗi MI-01~05 về naming không nhất quán, path phức tạp và comment style.
- Nhóm lỗi này không chặn workflow nhưng làm tài liệu khó bảo trì.

**Impact**

Agent mới hoặc người vận hành mới mất thêm thời gian khi trace rule/skill, nhất là ở nhóm clarification.

**Recommendation**

Sau khi xử lý CRITICAL/MAJOR, rà naming/path/comment style ở nhóm clarification và chuẩn hóa dần.

**Acceptance Criteria**

- Các reference path trong nhóm clarification dùng cùng convention.
- Comment/link style nhất quán với các skill còn lại.

---

## Summary

| Severity | Số lượng |
|---|---:|
| CRITICAL | 3 |
| MAJOR | 8 |
| MINOR | 2 |

---

## Handoff Table

| Finding ID | Severity | Repo/Module | Target File | Section/Anchor | Action | Required Change | Acceptance Criteria |
|---|---|---|---|---|---|---|---|
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/rules/review-handoff-policy.md` | `Nguồn report hợp lệ` | UPDATE | Cho phép xử lý cả section `## Fix Handoff For Target Agent` và `## Handoff Table`; ghi rõ `Handoff Table` là schema v2 hợp lệ. | Policy không bỏ qua report chỉ vì dùng section `Handoff Table`. |
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/agents/ba-agent-fix-agent.md` | `Bước 1 — Parse report` | UPDATE | Bổ sung alias mapping: `Repo/Module` -> `Target Agent System`, `Section/Anchor` -> `Target Section`, `Action` -> `Action Type`, `Required Change` -> `Fix Instruction`; chấp nhận `Repo/Module = ba-agent`. | Agent parse được cả schema legacy và schema v2. |
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/skills/shared/review-report-fix-handoff.md` | `Schema finding hợp lệ` | UPDATE | Mô tả schema v2 `Handoff Table` với 8 cột và rule map sang schema nội bộ hiện tại. | Skill document hóa schema v2 và backward compatibility với v1. |
| CRITICAL-002 | CRITICAL | ba-agent | `.claude/skills/clarification/requirement-clarification.md` | `References` | UPDATE | Sửa reference sai `@../rules/ba-conventions.md` thành path đúng tới `.claude/rules/ba-conventions.md`. | Skill load được rule nền ba-conventions. |
| CRITICAL-002 | CRITICAL | ba-agent | `.claude/skills/shared/resolve-oqs.md` | `References` | UPDATE | Sửa reference sai `@../rules/ba-conventions.md` thành path đúng tới `.claude/rules/ba-conventions.md`. | Skill load được rule nền ba-conventions. |
| CRITICAL-003 | CRITICAL | ba-agent | `AGENTS.md` | `Khi tác vụ tương ứng với một vai trò BA chuyên biệt` | UPDATE | Thêm `.claude/agents/ba-brainstorm-agent.md` và `.claude/agents/ba-orchestrator-agent.md` vào danh sách agent cần đọc. | Danh sách agent cần đọc khớp bảng subagent. |
| CRITICAL-003 | CRITICAL | ba-agent | `.claude/rules/project-context.md` | `Cấu trúc thư mục > agents` | UPDATE | Thêm `ba-orchestrator-agent.md` vào cây `.claude/agents/`. | Cây thư mục phản ánh đầy đủ agent đang tồn tại. |
| MAJOR-001 | MAJOR | ba-agent | `CLAUDE.md` | `Workflow 3 Phase > Phase 2` | UPDATE | Sửa mô tả Phase 2 từ song song sang tuần tự Wireframe -> React Prototype -> Feedback Triage; chỉ nhắc song song cho review độc lập nếu cần. | Không còn mâu thuẫn với `agent-workflow.md`. |
| MAJOR-001 | MAJOR | ba-agent | `GEMINI.md` | `Quy trình Phối hợp 3 Phase > Phase 2` | UPDATE | Sửa mô tả Phase 2 từ song song/phối hợp sang tuần tự theo `agent-workflow.md`. | Không còn mâu thuẫn với `agent-workflow.md`. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/rules/output-schema.md` | `Artifact và ID format` | UPDATE | Thêm `Traceability Map` với file `traceability-map.md`, trạng thái tự động sau Process Summary và prerequisite của SYNC. | Artifact table có Traceability Map. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/rules/output-schema.md` | `Traceability chain` | UPDATE | Cập nhật chain thành `... -> URD/SRS -> Process Summary -> Traceability Map`; ghi rõ map là nền tảng cho SYNC. | Chain khớp `AGENTS.md` và `agent-workflow.md`. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/skills/urd/document-integrity-check.md` | `Kiểm tra bổ sung — [Cần xác nhận]` | UPDATE | Bổ sung rule critical/non-critical confirmation và cách chuyển thành verdict. | Critical confirmation còn mở dẫn tới NEEDS FIX; non-critical có thể READY nhưng phải warning rõ. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/agents/ba-postcheck-agent.md` | `Output format / Phán quyết` | UPDATE | Cho phép output READY kèm warning hoặc NEEDS FIX theo rule critical confirmation. | Postcheck có verdict rõ khi còn `[Cần xác nhận]`. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/agents/urd-srs-agent.md` | `Nguyên tắc bất biến` | UPDATE | Sửa câu "không cần hỏi thêm" thành điều kiện chỉ đúng khi không còn critical `[Cần xác nhận]`. | URD/SRS agent không tự mâu thuẫn giữa allowed confirmation và Dev/Test ready. |
| MAJOR-004 | MAJOR | ba-agent | `.claude/skills/sync/change-handler.md` | `Bước 2 — Tìm REQ ID trong Traceability Map` | UPDATE | Không tự tạo REQ mới khi không match cho MODIFY/REMOVE; chỉ tạo khi ADD được BA xác nhận; thêm `match_status`. | Change Set không tạo duplicate REQ do match kém. |
| MAJOR-004 | MAJOR | ba-agent | `.claude/skills/sync/impact-analysis.md` | `Bước 1 — Parse thay đổi` | UPDATE | Nếu Change Set còn `AMBIGUOUS` thì dừng hỏi BA, không phân tích impact/patch. | SYNC không patch khi chưa xác định đúng REQ ID. |
| MAJOR-005 | MAJOR | ba-agent | `.claude/agents/ba-backlog-agent.md` | `Lưu file` | UPDATE | Chuẩn hóa output backlog sang `backlog-v[N].md` hoặc ghi rõ legacy `backlog.md` sẽ được version hóa ở lần refine/SYNC đầu tiên. | Backlog generate/refine/SYNC có cùng naming contract. |
| MAJOR-005 | MAJOR | ba-agent | `.claude/skills/sync/artifact-patch.md` | `Patch Backlog / Story / AC` | UPDATE | Đồng bộ target backlog với output của `ba-backlog-agent`; nếu hỗ trợ legacy thì quy định cách tìm `backlog.md` và tạo `backlog-v[N].md`. | Artifact patch không target file mà agent generate không tạo. |
| MAJOR-006 | MAJOR | ba-agent | `.claude/rules/project-context.md` | `Cấu trúc thư mục > output` | UPDATE | Thêm `brainstorm/` và `backlog/` vào cây output project. | Project context phản ánh đủ output Phase 0 và backlog. |
| MAJOR-007 | MAJOR | ba-agent | `.claude/rules/project-context.md` | `Cấu trúc thư mục > input` | UPDATE | Thêm `.claude/input/change-requests/` là inbox cho PO update/change request phục vụ SYNC. | SYNC input folder được document. |
| MAJOR-008 | MAJOR | ba-agent | `AGENTS.md` | `Skill Bắt Buộc Theo Tác Vụ` | UPDATE | Thêm `.claude/skills/diagram/SKILL.md` vào danh sách skill/task mapping cho yêu cầu vẽ diagram. | Agent biết phải load diagram skill khi có intent vẽ sơ đồ. |
| MAJOR-008 | MAJOR | ba-agent | `CLAUDE.md` | `Công cụ hệ thống` | UPDATE | Nhắc `diagram` skill hoặc trỏ rõ về `agent-workflow.md` cho intent vẽ diagram. | Root docs không bỏ sót diagram workflow. |
| MINOR-001 | MINOR | ba-agent | `CLAUDE.md` | `Rules chi tiết` | UPDATE | Thêm `.claude/rules/review-handoff-policy.md` vào danh sách rule chi tiết. | Claude docs nhắc đủ policy review/fix bridge. |
| MINOR-001 | MINOR | ba-agent | `GEMINI.md` | `ba-agent-fix-agent` row | UPDATE | Bổ sung `review-handoff-policy.md` là policy bắt buộc cùng `review-report-fix-handoff.md`. | Gemini docs không bỏ sót policy allowlist/ASK_CONFIRM. |
| MINOR-002 | MINOR | ba-agent | `.claude/skills/clarification/brainstorm.md` | `References / comment style` | UPDATE | Rà và chuẩn hóa path/comment style sau khi các CRITICAL/MAJOR đã xử lý. | Naming/path/comment style nhất quán hơn. |
| MINOR-002 | MINOR | ba-agent | `.claude/agents/ba-clarification-agent.md` | `Instruction style` | UPDATE | Rà comment/naming style và đồng bộ với convention chung nếu có lệch. | Agent prompt dễ bảo trì hơn. |
