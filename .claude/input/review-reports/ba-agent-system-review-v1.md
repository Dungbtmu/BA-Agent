# Agent Review Report — ba-agent System

**Ngày review:** 04/06/2026  
**Reviewer:** Agent Review Agent  
**Target Agent System:** ba-agent  
**Report ID:** BA-AGENT-SYSTEM-REVIEW-V1  
**Verdict:** NEEDS FIX  

---

## Review Scope

Review toàn bộ cấu trúc vận hành chính của `ba-agent`, bao gồm:

- Entrypoint và orchestration: `AGENTS.md`
- Rule nền: `.claude/rules/*.md`
- Agent prompt: `.claude/agents/*.md`
- Skill prompt: `.claude/skills/*.md`
- Review handoff bridge: `.claude/input/review-reports/`, `ba-agent-fix-agent`, `review-handoff-policy`, `review-report-fix-handoff`

Không review chất lượng nội dung nghiệp vụ trong `.claude/output/` vì đây là artifact dự án, không phải prompt/rule vận hành agent.

---

## Findings

### CRITICAL-001 — Contract giữa Agent Review Agent và ba-agent-fix-agent đang lệch schema

**Evidence**

- `.claude/agents/ba-agent-fix-agent.md` yêu cầu section `Fix Handoff For Target Agent` có 8 trường, trong đó có `Severity`.
- `.claude/skills/review-report-fix-handoff.md` cũng mô tả schema có `Severity`.
- Trong khi contract export hiện tại của Agent Review Agent đang hướng tới bảng handoff không bắt buộc `Severity` như một cột riêng. Nếu report chỉ có `Finding ID` dạng `CRITICAL-001`, `MAJOR-001`, `MINOR-001` thì `ba-agent-fix-agent` có thể parse thiếu trường hoặc sort severity sai.

**Impact**

Luồng tự động "Agent Review Agent export report -> ba-agent đọc report -> sửa theo report" có nguy cơ không chạy ổn định. Đây là lỗi contract giữa producer và consumer, ảnh hưởng trực tiếp mục tiêu kết nối hai agent.

**Recommendation**

Cho `ba-agent-fix-agent` và `review-report-fix-handoff` chấp nhận 2 dạng schema:

- Dạng 8 cột có `Severity`
- Dạng 7 cột không có `Severity`, tự derive severity từ prefix của `Finding ID`

---

### MAJOR-001 — Phạm vi apply handoff đang quá hẹp, không sửa được root system files

**Evidence**

- `.claude/rules/review-handoff-policy.md` chỉ cho sửa file trong `.claude/agents/`, `.claude/skills/`, `.claude/rules/`.
- Trong thực tế, nhiều lỗi vận hành agent nằm ở root files như `AGENTS.md`, `README.md`, `CLAUDE.md`, `GEMINI.md`.
- `AGENTS.md` đang là entrypoint bắt buộc, nhưng policy hiện tại khiến `ba-agent-fix-agent` không được phép sửa chính entrypoint nếu review report target vào đó.

**Impact**

Review report có thể phát hiện lỗi quan trọng ở entrypoint nhưng fix agent sẽ bỏ qua hoặc không được phép sửa. Bridge review/fix vì vậy không self-contained.

**Recommendation**

Mở rộng allowlist Target File cho:

- `AGENTS.md`
- `README.md`
- `CLAUDE.md`
- `GEMINI.md`
- `.claude/agents/`
- `.claude/skills/`
- `.claude/rules/`

---

### MAJOR-002 — Quy ước output path còn không nhất quán giữa `.claude/output/` và `output/`

**Evidence**

Một số file vẫn dùng pattern `output/[tên_dự_án]/` hoặc `output/[dự_án]/` trong khi entrypoint chuẩn là `.claude/output/[tên_dự_án]/`:

- `AGENTS.md`
- `.claude/rules/agent-workflow.md`
- `.claude/agents/ba-research-agent.md`
- `.claude/agents/ba-wireframe-agent.md`
- `.claude/agents/ui-react-agent.md`
- `.claude/skills/process-log.md`
- `.claude/skills/wireframe-design-system.md`
- `.claude/skills/react-ui-generation.md`

**Impact**

Agent có thể lưu output sai thư mục, làm mất traceability hoặc khiến các phase sau không tìm thấy artifact mới nhất.

**Recommendation**

Chuẩn hóa toàn bộ hướng dẫn output path sang `.claude/output/[tên_dự_án]/...`. Nếu cần nhắc legacy pattern thì phải ghi rõ là "pattern cũ chỉ dùng khi thư mục dự án hiện hữu đang theo pattern đó".

---

### MAJOR-003 — Phase 2 còn mâu thuẫn về quan hệ Wireframe và React Prototype

**Evidence**

- `AGENTS.md` mô tả Phase 2 theo thứ tự `Wireframe -> React Prototype -> Feedback Triage -> Refine UI`.
- `.claude/rules/agent-workflow.md` vẫn có đoạn mô tả Phase 2 theo kiểu song song/2 chiều giữa Wireframe và React Prototype.
- `ui-react-agent` lại yêu cầu tạo prototype dựa trên wireframe/solution đã chốt.

**Impact**

Agent chính có thể hiểu sai rằng Wireframe và React Prototype chạy song song ngay từ đầu, trong khi React Prototype phụ thuộc vào wireframe hoặc solution đã đủ rõ.

**Recommendation**

Chuẩn hóa Phase 2 thành:

`Wireframe -> React Prototype -> Feedback Triage -> Refine đúng phần feedback`

Chỉ cho phép chạy song song ở các bước review độc lập, không phải bước tạo React prototype khi chưa có wireframe/solution đủ rõ.

---

### MAJOR-004 — Rule về `[Cần xác nhận]` trong postcheck/document integrity còn tự mâu thuẫn

**Evidence**

- `.claude/skills/document-integrity-check.md` có đoạn xem `[Cần xác nhận: ...]` là WARN nếu đúng chỗ.
- Cùng file vẫn có đoạn khác coi `[Cần xác nhận: ...]` còn tồn tại là FAIL.
- `.claude/agents/ba-postcheck-agent.md` đang theo hướng WARN, không phải FAIL.

**Impact**

Postcheck có thể trả verdict khác nhau tùy đọc đoạn nào trước: một bên cho phép URD/SRS còn mục cần xác nhận, bên kia chặn handoff vì placeholder còn tồn tại.

**Recommendation**

Chốt một rule thống nhất:

- Trong URD/SRS: `[Cần xác nhận: ...]` được phép nhưng phải đưa vào danh sách cần xác nhận, severity WARN.
- Trong tài liệu handoff cuối cho Dev/Test: nếu còn mục critical chưa xác nhận thì NEEDS FIX hoặc BLOCK tùy mức độ.

---

### MAJOR-005 — `project-context.md` chưa phản ánh các thành phần mới của review/fix bridge

**Evidence**

`.claude/rules/project-context.md` vẫn mô tả cấu trúc cũ, chưa liệt kê:

- `.claude/agents/ba-agent-fix-agent.md`
- `.claude/rules/review-handoff-policy.md`
- `.claude/skills/review-report-fix-handoff.md`
- `.claude/input/review-reports/`

**Impact**

Khi Codex/agent đọc project context trước, hệ thống review/fix bridge có thể bị xem là ngoại lệ thay vì thành phần chính thức của BA Agent System.

**Recommendation**

Cập nhật cây thư mục và mô tả vai trò bridge vào `project-context.md`.

---

### MINOR-001 — README chưa đủ làm entrypoint phụ cho người vận hành

**Evidence**

`README.md` hiện rất ngắn, chưa mô tả:

- Entry point chính là `AGENTS.md`
- Review report inbox nằm ở `.claude/input/review-reports/`
- Cách kích hoạt `ba-agent-fix-agent`
- Luồng Agent Review Agent -> ba-agent-fix-agent -> Fix Summary

**Impact**

Người vận hành mới có thể phải mở nhiều file mới hiểu cách connect hai agent.

**Recommendation**

Thêm phần ngắn trong README về cấu trúc, entrypoint và review/fix bridge.

---

### MINOR-002 — Ví dụ path giả trong Fix Summary dễ gây nhiễu khi kiểm tra reference

**Evidence**

`.claude/agents/ba-agent-fix-agent.md` dùng ví dụ:

- `.claude/agents/xxx.md`
- `.claude/skills/yyy.md`
- `.claude/rules/zzz.md`

Các path này nhìn giống path thật và có thể bị script kiểm tra reference đánh dấu missing file.

**Impact**

Không ảnh hưởng runtime chính, nhưng gây nhiễu khi audit integrity tự động.

**Recommendation**

Đổi ví dụ sang placeholder không giống path thật, ví dụ `<target-agent-file>`, `<target-skill-file>`, `<target-rule-file>`.

---

## Effectiveness Scorecard

| Tiêu chí | Điểm | Nhận xét |
|---|---:|---|
| Scope clarity | 8/10 | Phạm vi BA rõ, có phase và agent mapping đầy đủ. |
| Routing quality | 7/10 | Có mapping agent/skill tốt, nhưng Phase 2 còn mâu thuẫn. |
| Schema quality | 5/10 | Handoff bridge đã có, nhưng contract producer/consumer lệch. |
| Evidence discipline | 8/10 | Nhiều rule cấm bịa, yêu cầu assumption/risk/traceability rõ. |
| Maintainability | 6/10 | Một số path cũ và file context chưa sync khiến hệ thống dễ drift. |
| Automation readiness | 5/10 | Có fix-agent, nhưng schema và allowlist hiện chặn một phần auto-fix. |

**Overall:** 6.5/10  
**Kết luận:** Hệ thống đã có nền tốt, nhưng chưa nên coi review/fix bridge là ổn định cho đến khi sửa các lỗi CRITICAL/MAJOR ở trên.

---

## Quality Gate

| Gate | Status | Note |
|---|---|---|
| Scope | PASS | BA Agent System có phạm vi rõ. |
| Routing | NEEDS FIX | Phase 2 còn mâu thuẫn về dependency. |
| Schema | FAIL | Handoff schema giữa review agent và fix agent chưa khớp. |
| Verdict mapping | PASS | CRITICAL/MAJOR/MINOR được dùng nhất quán ở review flow. |
| README/Structure | NEEDS FIX | README và project-context chưa cập nhật đủ bridge mới. |
| Evidence | PASS | Findings có file/behavior cụ thể. |

---

## Fix Handoff For Target Agent

| Finding ID | Severity | Target Agent System | Target File | Target Section | Action Type | Fix Instruction | Acceptance Criteria |
|---|---|---|---|---|---|---|---|
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/agents/ba-agent-fix-agent.md` | `Bước 1 — Parse report` | UPDATE | Cập nhật hướng dẫn parse để chấp nhận cả schema 8 cột có `Severity` và schema 7 cột không có `Severity`; nếu thiếu `Severity`, derive từ prefix của `Finding ID` (`CRITICAL-`, `MAJOR-`, `MINOR-`). | Agent không SKIPPED finding chỉ vì thiếu cột `Severity` khi `Finding ID` đã chứa severity prefix. |
| CRITICAL-001 | CRITICAL | ba-agent | `.claude/skills/review-report-fix-handoff.md` | `Schema finding hợp lệ` | UPDATE | Mô tả rõ `Severity` là optional nếu `Finding ID` đã có prefix severity; bổ sung rule derive severity và sort theo severity đã derive. | Skill parse được cả 7-field và 8-field handoff table, không mâu thuẫn với Agent Review Agent export. |
| MAJOR-001 | MAJOR | ba-agent | `.claude/rules/review-handoff-policy.md` | `Phạm vi sửa` | UPDATE | Mở rộng allowlist Target File cho root files `AGENTS.md`, `README.md`, `CLAUDE.md`, `GEMINI.md` ngoài `.claude/agents/`, `.claude/skills/`, `.claude/rules/`. | Finding target vào root entrypoint/system docs không bị policy chặn. |
| MAJOR-001 | MAJOR | ba-agent | `.claude/agents/ba-agent-fix-agent.md` | `Quy trình apply` | UPDATE | Bổ sung pre-flight validation theo allowlist mới và ghi rõ root system files hợp lệ khi Target Agent System là `ba-agent`. | Pre-flight không đánh dấu SKIPPED với `AGENTS.md`, `README.md`, `CLAUDE.md`, `GEMINI.md` nếu file tồn tại. |
| MAJOR-002 | MAJOR | ba-agent | `AGENTS.md` | `BA Workflow` | UPDATE | Chuẩn hóa ví dụ version path từ `output/v1/`, `output/v2/` sang `.claude/output/v1/`, `.claude/output/v2/` hoặc ghi rõ chỉ là legacy pattern nếu dự án hiện hữu đã dùng. | Không còn hướng dẫn mơ hồ khiến output có thể lưu ngoài `.claude/output/`. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/rules/agent-workflow.md` | `BA Workflow` | UPDATE | Thay các path `output/[tên_dự_án]/` hoặc `output/[dự_án]/` bằng `.claude/output/[tên_dự_án]/`. | `rg "output/\\[" .claude/rules/agent-workflow.md` không còn trả về path thiếu `.claude/`, trừ khi được ghi rõ là legacy. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/agents/ba-research-agent.md` | `Output bắt buộc` | UPDATE | Chuẩn hóa output domain brief sang `.claude/output/[tên_dự_án]/research/domain-brief.md`. | Research agent không còn hướng dẫn lưu vào `output/` root. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/agents/ba-wireframe-agent.md` | `Design guideline` | UPDATE | Chuẩn hóa design guideline path sang `.claude/output/[tên_dự_án]/design-guideline.md`. | Wireframe agent tìm guideline đúng theo cấu trúc `.claude/output`. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/agents/ui-react-agent.md` | `Design guideline` | UPDATE | Chuẩn hóa design guideline path sang `.claude/output/[tên_dự_án]/design-guideline.md`. | UI React agent tìm guideline đúng theo cấu trúc `.claude/output`. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/skills/process-log.md` | `Output` | UPDATE | Thay toàn bộ pattern `output/[dự_án]/...` bằng `.claude/output/[tên_dự_án]/...`. | Process summary artifact không còn hướng dẫn lưu sai thư mục. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/skills/wireframe-design-system.md` | `Design guideline override` | UPDATE | Thay path `output/[tên_dự_án]/design-guideline.md` bằng `.claude/output/[tên_dự_án]/design-guideline.md`. | Skill wireframe dùng cùng path với AGENTS.md. |
| MAJOR-002 | MAJOR | ba-agent | `.claude/skills/react-ui-generation.md` | `Design guideline` | UPDATE | Thay path `output/[tên_dự_án]/design-guideline.md` bằng `.claude/output/[tên_dự_án]/design-guideline.md`. | Skill React prototype dùng cùng path với AGENTS.md. |
| MAJOR-003 | MAJOR | ba-agent | `.claude/rules/agent-workflow.md` | `Phase 2` | UPDATE | Chuẩn hóa Phase 2 thành tuần tự `Wireframe -> React Prototype -> Feedback Triage -> Refine UI`; chỉ ghi song song cho các bước review độc lập, không cho tạo React trước khi có wireframe/solution đủ rõ. | Không còn mô tả Wireframe và React Prototype như hai nhánh song song phụ thuộc lẫn nhau. |
| MAJOR-004 | MAJOR | ba-agent | `.claude/skills/document-integrity-check.md` | `Placeholder / Cần xác nhận` | UPDATE | Đồng bộ rule `[Cần xác nhận: ...]`: trong URD/SRS được phép với severity WARN nếu có danh sách cần xác nhận; chỉ FAIL/BLOCK khi là thông tin critical cho handoff dev/test mà chưa được xử lý. | File không còn một đoạn coi `[Cần xác nhận]` là WARN và đoạn khác coi là FAIL tuyệt đối. |
| MAJOR-005 | MAJOR | ba-agent | `.claude/rules/project-context.md` | `Cấu trúc thư mục` | UPDATE | Bổ sung `ba-agent-fix-agent.md`, `review-handoff-policy.md`, `review-report-fix-handoff.md` và `.claude/input/review-reports/` vào mô tả cấu trúc. | Project context phản ánh đầy đủ review/fix bridge hiện tại. |
| MINOR-001 | MINOR | ba-agent | `README.md` | `Overview` | ADD | Thêm mô tả ngắn về entrypoint `AGENTS.md`, inbox `.claude/input/review-reports/`, cách dùng `ba-agent-fix-agent`, và output Fix Summary. | Người mới đọc README hiểu cách vận hành bridge mà không cần dò toàn bộ file. |
| MINOR-002 | MINOR | ba-agent | `.claude/agents/ba-agent-fix-agent.md` | `Output — Fix Summary` | UPDATE | Đổi ví dụ path giả `.claude/agents/xxx.md`, `.claude/skills/yyy.md`, `.claude/rules/zzz.md` thành placeholder không giống path thật, ví dụ `<target-agent-file>`. | Reference check không còn báo missing file vì ví dụ path giả. |

---

## Handoff Export Metadata

**Export status:** EXPORTED  
**Export target:** `.claude/input/review-reports/ba-agent-system-review-v1.md`  
**Target agent expected to consume:** `ba-agent-fix-agent`  
**Apply mode expected:** BA xác nhận trước khi apply theo pre-flight của `ba-agent-fix-agent`  
**Do not auto-apply:** TRUE  

---

## Missing Instructions / Open Questions

1. Có muốn `ba-agent-fix-agent` tự apply `UPDATE/ADD` sau pre-flight một lần, hay mỗi finding vẫn cần BA confirm riêng?
2. Có muốn root files `CLAUDE.md` và `GEMINI.md` được sync từ `AGENTS.md` tự động, hay chỉ xem là docs tham chiếu?
3. Có muốn tạo thêm rule versioning riêng cho review report và fix summary không?

---

## Summary

`ba-agent` đã có cấu trúc agent/skill/rule khá đầy đủ và đã bắt đầu có bridge để nhận report từ Agent Review Agent. Tuy nhiên, bridge chưa thật sự ổn định vì schema handoff lệch và policy sửa file còn chặn chính các root entrypoint quan trọng. Nên sửa `CRITICAL-001`, `MAJOR-001`, `MAJOR-002`, `MAJOR-003`, `MAJOR-004`, `MAJOR-005` trước khi dùng `ba-agent-fix-agent` như một bước tự động trong quy trình.
