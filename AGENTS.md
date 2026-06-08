# AGENTS.md

Repo này là **BA Agent System** dạng tài liệu Markdown, dùng để hỗ trợ BA tạo và refine tài liệu phân tích nghiệp vụ cho nhiều dự án/domain khác nhau. Repo không có ứng dụng để build, test hoặc compile.

Sử dụng file này làm entrypoint cho Codex khi vận hành BA Agent System. Các file trong `.claude/` vẫn là nguồn rule, agent và skill chính; Codex phải đọc đúng file liên quan trước khi xử lý tác vụ BA.

## Cấu Trúc Workspace

- `.claude/rules/` — rule bắt buộc, áp dụng cho mọi session BA.
- `.claude/agents/` — prompt định nghĩa vai trò BA/UI chuyên biệt.
- `.claude/skills/` — kỹ năng hỗ trợ research domain, phân tích, làm rõ, viết URD/SRS, sơ đồ URD/SRS, wireframe design system, feedback triage và React prototype.
- `.claude/input/` — tài liệu đầu vào như template URD, PRD, PDF nghiệp vụ, ghi chú thô.
- `.claude/output/[tên_dự_án]/` — tài liệu đầu ra theo từng dự án, ví dụ `.claude/output/cvm/`.

Input có thể là mô tả thô, bullet points, PRD, URD, PDF, wireframe, backlog hoặc output từ agent khác. Khi input là PDF hoặc tài liệu không đọc được trực tiếp, cố gắng trích xuất nội dung nếu công cụ hỗ trợ; nếu không, hỏi user cung cấp bản text/Markdown.

## Rule Bắt Buộc

Trước khi tạo hoặc review tài liệu BA, đọc và áp dụng các file sau:

- `.claude/rules/project-context.md` — bối cảnh BA Agent System, phạm vi BA, cấu trúc thư mục
- `.claude/rules/ba-persona.md` — tư duy và phong cách Senior BA
- `.claude/rules/language.md` — yêu cầu về ngôn ngữ tiếng Việt
- `.claude/rules/ba-conventions.md` — quy tắc chung cho mọi skill: IT-BA framing, no-re-ask, assumption, approval gate (L1/L2/L3), OQ format
- `.claude/rules/agent-workflow.md` — pipeline Phase 1-2-3, intent recognition, chế độ GENERATE/REFINE/REVIEW
- `.claude/rules/output-schema.md` — danh sách artifact, ID format, traceability chain
- `.claude/rules/review-handoff-policy.md` — quy tắc phạm vi sửa khi apply review report từ Agent Review Agent

Khi tác vụ tương ứng với một vai trò BA chuyên biệt, đọc thêm file phù hợp trong `.claude/agents/`:

- `.claude/agents/ba-research-agent.md`
- `.claude/agents/ba-clarification-agent.md`
- `.claude/agents/ba-solution-agent.md`
- `.claude/agents/ba-devil-advocate-agent.md`
- `.claude/agents/ba-backlog-agent.md`
- `.claude/agents/ba-wireframe-agent.md`
- `.claude/agents/ui-react-agent.md`
- `.claude/agents/ui-feedback-agent.md`
- `.claude/agents/urd-srs-agent.md`
- `.claude/agents/ba-qa-agent.md`
- `.claude/agents/ba-postcheck-agent.md`
- `.claude/agents/ba-process-summary-agent.md`
- `.claude/agents/ba-agent-fix-agent.md`

## Chế Độ Vận Hành

Với yêu cầu BA mới, đi theo 3 phase sau trừ khi user yêu cầu rõ một phạm vi hẹp hơn:

```text
Phase 1: Clarification -> Solution -> [Devil's Advocate] -> [Backlog/Epic/User Story/AC nếu BA yêu cầu]
Phase 2: Wireframe -> React Prototype -> Feedback Triage -> Refine UI
Phase 3: URD/SRS -> QA Review -> Refine đúng section có issue -> Post-check -> Process Summary
```

Nguyên tắc xử lý input thiếu:

- Với tác vụ BA thông thường: nêu assumption rõ ràng và tiếp tục dựa trên thông tin hiện có.
- Với thông tin critical như Actors, phân quyền, quy trình chính: hỏi lại user trước khi viết tài liệu hoàn chỉnh.
- Không bịa nghiệp vụ, không tự tạo chi tiết kỹ thuật không có trong input.
- Output chính của hệ thống là URD/SRS. Epic, User Story và Acceptance Criteria là artifact phụ trợ, chỉ tạo khi BA yêu cầu rõ hoặc khi cần làm rõ yêu cầu trước khi viết URD/SRS.
- Có thể bắt đầu từ bất kỳ phase nào nếu user đã có sẵn input phù hợp, ví dụ đã có wireframe thì bắt đầu từ URD/SRS.

## Quy Ước Chạy Subagent Trong Codex

Các file `.claude/agents/*.md` là prompt định nghĩa vai trò BA/UI chuyên biệt. Trong Codex, các file này không tự động trở thành agent độc lập. Khi user yêu cầu rõ việc chạy subagent, chạy nhiều agent, chạy song song, parallel review hoặc full BA agent flow bằng subagents, Codex phải tạo các subagent độc lập theo mapping dưới đây.

| Subagent | File hướng dẫn | Phạm vi chịu trách nhiệm |
|---|---|---|
| `ba-research-agent` | `.claude/agents/ba-research-agent.md` | Research domain mới, tạo Domain Brief khi BA chưa biết domain hoặc cần bối cảnh trước khi phân tích |
| `ba-clarification-agent` | `.claude/agents/ba-clarification-agent.md` | Làm rõ requirement, xác định missing information, assumptions, risks và câu hỏi cho stakeholder |
| `ba-solution-agent` | `.claude/agents/ba-solution-agent.md` | Đề xuất solution, user flow, edge cases, dependencies và trade-offs |
| `ba-devil-advocate-agent` | `.claude/agents/ba-devil-advocate-agent.md` | Phản biện chéo solution từ 4 góc nhìn độc lập (User / PO / Dev / Risk) trước khi vào Phase 2 |
| `ba-backlog-agent` | `.claude/agents/ba-backlog-agent.md` | Chia Epic, User Stories, Acceptance Criteria và dependencies khi BA yêu cầu |
| `ba-wireframe-agent` | `.claude/agents/ba-wireframe-agent.md` | Phác thảo màn hình, layout, UI flow, component, state và interaction |
| `ui-react-agent` | `.claude/agents/ui-react-agent.md` | Tạo React prototype để xem trực quan và lấy feedback giao diện |
| `ui-feedback-agent` | `.claude/agents/ui-feedback-agent.md` | Triage feedback UI, phân loại góp ý và chỉ định sửa đúng phần cần sửa |
| `urd-srs-agent` | `.claude/agents/urd-srs-agent.md` | Viết tài liệu URD/SRS đầy đủ, dev/test ready |
| `ba-qa-agent` | `.claude/agents/ba-qa-agent.md` | Review chất lượng, phát hiện issue, kiểm tra traceability và consistency |
| `ba-postcheck-agent` | `.claude/agents/ba-postcheck-agent.md` | Audit cấu trúc, traceability, naming và version hygiene sau khi ba-qa-agent chấp thuận content |
| `ba-process-summary-agent` | `.claude/agents/ba-process-summary-agent.md` | Tổng kết quá trình: tạo Decision Log, Assumption Register và Handoff Note cho Dev/Tester |
| `ba-orchestrator-agent` | `.claude/agents/ba-orchestrator-agent.md` | Điều phối toàn bộ BA pipeline (GENERATE) hoặc đồng bộ artifact khi requirement thay đổi (SYNC); dispatch agent con, đọc output, dừng tại checkpoint cần BA confirm |
| `ba-agent-fix-agent` | `.claude/agents/ba-agent-fix-agent.md` | Apply review report từ Agent Review Agent: đọc report tại `.claude/input/review-reports/`, convert thành patch plan, sửa đúng file, báo cáo kết quả |

Nguyên tắc điều phối:

- Agent chính chịu trách nhiệm đọc `AGENTS.md`, các rule trong `.claude/rules/`, điều phối subagent và tổng hợp kết quả cuối cùng.
- Mỗi subagent chỉ xử lý đúng phạm vi vai trò của mình.
- Khi chạy tuần tự, output của subagent trước là input của subagent sau.
- Khi chạy song song, chỉ tách các việc không phụ thuộc trực tiếp vào nhau, ví dụ review backlog, review wireframe và kiểm tra traceability.
- Nếu subagent phát hiện thông tin thiếu hoặc mâu thuẫn nghiêm trọng, agent chính phải tổng hợp lại thành câu hỏi hoặc finding trước khi tiếp tục.
- Không dùng subagent để bỏ qua quy trình bắt buộc trong `BA Workflow`.
- `ba-backlog-agent` là optional. Không tự tạo Epic/User Story/AC nếu BA không yêu cầu và các thông tin hiện có đã đủ để đi tiếp.
- `ba-devil-advocate-agent` là optional nhưng khuyến nghị sau `ba-solution-agent`. Nếu phán quyết là BLOCK, quay lại `ba-solution-agent` sửa đúng điểm bị challenge; không làm lại toàn bộ solution.
- Sau khi `urd-srs-agent` viết URD/SRS xong, tự động dùng `ba-qa-agent` review; nếu còn `CRITICAL` thì quay lại sửa đúng section có vấn đề, không viết lại toàn bộ.
- Sau khi `ba-qa-agent` chốt tài liệu, tự động chạy `ba-postcheck-agent` audit cấu trúc; nếu NEEDS FIX thì sửa đúng chỗ và chạy lại.
- Sau khi `ba-postcheck-agent` báo READY FOR HANDOFF, chạy `ba-process-summary-agent` để tạo Decision Log, Assumption Register và Handoff Note.

Ví dụ user prompt có thể kích hoạt subagent:

```text
Dùng BA subagents song song để review bộ tài liệu này.
```

```text
Chạy full BA agent flow bằng subagents độc lập.
```

```text
Tạo backlog từ tài liệu này, dùng ba-clarification-agent trước; nếu đủ rõ thì chuyển qua ba-backlog-agent.
```

```text
Từ wireframe và backlog hiện có, dùng urd-srs-agent để viết URD/SRS.
```

```text
Tôi chưa biết domain này, dùng ba-research-agent tạo Domain Brief trước.
```

```text
Tạo React prototype từ wireframe và dùng ui-feedback-agent để xử lý feedback vòng sau.
```

## Skill Bắt Buộc Theo Tác Vụ

Khi agent hoặc task yêu cầu skill, phải đọc skill tương ứng trước khi viết output:

- `.claude/skills/input-analysis.md` — đọc và trích xuất requirement từ tài liệu có sẵn (PRD, email, ghi chú); phân loại rõ/chưa rõ/mâu thuẫn
- `.claude/skills/requirement-clarification.md` — làm rõ requirement
- `.claude/skills/problem-framing.md` — đóng khung bài toán
- `.claude/skills/domain-research.md` — research domain mới, tổng hợp bối cảnh và thuật ngữ nghiệp vụ
- `.claude/skills/domain-gap-analysis.md` — so sánh domain điển hình (từ Domain Brief) với yêu cầu thực tế của client; output là danh sách gap CRITICAL/MAJOR/MINOR để định hướng câu hỏi clarify
- `.claude/skills/as-is-analysis.md` — phân tích hiện trạng thực tế dự án (hệ thống cũ, quy trình đang chạy, constraint); xác định đã biết gì và còn mở gì để thu hẹp phạm vi Clarification
- `.claude/skills/stakeholder-mapping.md` — xác định stakeholder
- `.claude/skills/context-constraint-analysis.md` — phân tích context và ràng buộc
- `.claude/skills/assumption-risk-analysis.md` — xác định assumption và risk
- `.claude/skills/user-persona-identification.md` — xác định user/persona
- `.claude/skills/wireframe-design-system.md` — chuẩn thiết kế chung cho wireframe và React prototype; project có thể override bằng `design-guideline.md`
- `.claude/skills/react-ui-generation.md` — tạo React prototype từ wireframe/solution
- `.claude/skills/ui-feedback-triage.md` — phân loại feedback UI và xác định phạm vi sửa
- `.claude/skills/urd-srs-structure.md` — cấu trúc chuẩn tài liệu URD/SRS
- `.claude/skills/urd-workflow-diagram.md` — tạo Workflow Diagram/Swimlane và bảng diễn giải cho URD/SRS section II.1
- `.claude/skills/urd-function-tree.md` — tạo Business Function Diagram/cây phân cấp chức năng cho URD/SRS section II.2
- `.claude/skills/urd-permission-matrix.md` — tạo Permission Matrix và RBAC Matrix cho URD/SRS section II.3 và II.4
- `.claude/skills/urd-sequence-diagram.md` — tạo Sequence Diagram và bảng diễn giải cho URD/SRS section II.5
- `.claude/skills/urd-use-case.md` — viết Use Case Specification chi tiết cho URD/SRS section III
- `.claude/skills/urd-screen-spec.md` — viết đặc tả giao diện chức năng cho URD/SRS section IV
- `.claude/skills/urd-review-checklist.md` — checklist review chất lượng URD/SRS trước khi chốt
- `.claude/skills/solution-critique.md` — framework phản biện solution 4 lens (User / Business / Feasibility / Risk)
- `.claude/skills/document-integrity-check.md` — kiểm tra cấu trúc, traceability, naming và version hygiene của tài liệu
- `.claude/skills/process-log.md` — trích xuất decision, assumption, scope delta và tạo handoff note
- `.claude/skills/traceability-map.md` — xây dựng và duy trì Traceability Map; link REQ → UC → WF → URD Section → Story
- `.claude/skills/impact-analysis.md` — tính artifact bị ảnh hưởng khi requirement thay đổi; output Impact Report cho orchestrator
- `.claude/skills/change-handler.md` — parse trigger thay đổi từ BA mô tả hoặc file PO mới; output Change Set chuẩn hóa
- `.claude/skills/artifact-patch.md` — patch đúng phần bị ảnh hưởng trong từng artifact theo Impact Report; không viết lại toàn bộ
- `.claude/skills/resolve-oqs.md` — collect + resolve Open Questions sau mỗi Write doc; cascade scan section bị ảnh hưởng; chạy trước suggest downstream

Mapping bắt buộc:

- `ba-research-agent` phải đọc `domain-research`.
- `ba-clarification-agent` chỉ cần đọc `requirement-clarification` — skill này là orchestrator, tự quyết định gọi `input-analysis`, `as-is-analysis`, `domain-gap-analysis`, `problem-framing` theo đúng thứ tự dựa trên context.
- `ba-solution-agent` phải đọc `user-persona-identification`, `stakeholder-mapping`, `assumption-risk-analysis`, `context-constraint-analysis`. Đây là agent duy nhất chạy `stakeholder-mapping` và `user-persona-identification` — không chạy ở phase Clarification.
- `ba-backlog-agent` phải đọc `requirement-clarification` và `user-persona-identification`.
- `ba-wireframe-agent` phải đọc `wireframe-design-system` và `ui-feedback-triage`.
- `ui-react-agent` phải đọc `wireframe-design-system` và `react-ui-generation`.
- `ui-feedback-agent` phải đọc `ui-feedback-triage`.
- `urd-srs-agent` phải đọc `urd-srs-structure` trước, sau đó đọc các skill section-level tương ứng: `urd-workflow-diagram`, `urd-function-tree`, `urd-permission-matrix`, `urd-sequence-diagram`, `urd-use-case`, `urd-screen-spec`.
- `ba-qa-agent` phải đọc `urd-review-checklist`, `assumption-risk-analysis` và `requirement-clarification`.
- `ba-devil-advocate-agent` phải đọc `solution-critique` và `assumption-risk-analysis`.
- `ba-postcheck-agent` phải đọc `document-integrity-check`.
- `ba-process-summary-agent` phải đọc `process-log` và `assumption-risk-analysis`.
- `ba-agent-fix-agent` phải đọc `review-handoff-policy` (rules) và `review-report-fix-handoff` (skill).
- `ba-orchestrator-agent` phải đọc `traceability-map`, `impact-analysis`, `change-handler`, `artifact-patch` khi chạy chế độ SYNC.

## BA Workflow

Với tác vụ `GENERATE`:

```text
Phase 1 — Làm rõ & Giải pháp:
Input -> [Research domain nếu cần] -> Clarification -> Solution -> [Epic/User Story/AC nếu BA yêu cầu]

Phase 2 — Giao diện:
Wireframe -> React prototype -> Feedback triage -> Refine đúng phần được góp ý -> Lặp đến khi chốt

Phase 3 — Tài liệu:
URD/SRS -> QA Review -> Refine đúng section có issue -> Lặp đến khi không còn CRITICAL
-> Post-check -> Process Summary -> [T] Tạo Traceability Map
```

Với tác vụ `SYNC` (requirement thay đổi sau khi đã có Traceability Map):

```text
[S1] change-handler  -> parse trigger (BA mô tả / file PO mới) -> Delta Summary -> BA confirm
[S2] impact-analysis -> tra Traceability Map -> Impact Report (danh sách artifact + thứ tự patch)
[S3] artifact-patch  -> patch đúng phần bị ảnh hưởng -> Patch Summary
[S4] ba-qa-agent     -> verify không có conflict mới
[S5] ba-postcheck-agent -> audit cấu trúc sau patch
```

Điều kiện SYNC: Traceability Map phải tồn tại tại `.claude/output/[tên_dự_án]/traceability-map.md`. Nếu chưa có, phải tạo từ URD/SRS hiện tại trước khi SYNC.

- URD/SRS là output chính.
- Epic, User Story và AC là optional; chỉ tạo khi BA yêu cầu rõ hoặc khi agent chính xác định thật sự cần để làm rõ yêu cầu.
- Không chặn workflow chỉ vì thiếu PRD chuẩn; chấp nhận mô tả thô, ghi chú rời, bullet points, PRD, wireframe, backlog hoặc output từ agent khác.
- Không bịa thông tin kỹ thuật như Architecture, API, Data, Event, ERD chi tiết hoặc infrastructure.
- Đầu ra mặc định lưu tại `.claude/output/[tên_dự_án]/`; nếu chưa xác định dự án thì hỏi user hoặc dùng tên thư mục phù hợp với input.
- Với các vòng lặp version, ưu tiên pattern hiện có trong thư mục dự án, ví dụ `.claude/output/[tên_dự_án]/v1/`, `.claude/output/[tên_dự_án]/v2/` hoặc `wireframe-v[N].md`, `urd-srs-v[N].md`. Nếu dự án hiện hữu đang dùng pattern cũ `output/v1/` thì giữ nguyên pattern đó và chỉ tăng version theo cách đang dùng.

Với tác vụ `REFINE`:

- Chỉ sửa đúng phần được yêu cầu.
- Không tái tạo toàn bộ tài liệu.
- Giữ nguyên nội dung không liên quan.
- Áp dụng cho wireframe, React prototype, URD/SRS và các artifact phụ trợ.
- Tăng version theo pattern hiện có, ví dụ `wireframe-v3.md` -> `wireframe-v4.md`, `lovable-prompt-v3.md` -> `lovable-prompt-v4.md`.
- Không ghi đè file version cũ trừ khi user yêu cầu rõ.

Với tác vụ `REVIEW`:

- Ưu tiên liệt kê findings.
- Phân loại từng vấn đề theo `CRITICAL`, `MAJOR`, hoặc `MINOR`.
- Mỗi vấn đề phải có khuyến nghị cụ thể, có thể hành động được.
- Nếu có `CRITICAL`, phải sửa trước khi chuyển phase tiếp theo hoặc chốt tài liệu.
- Sau review, quay lại đúng section/bước có vấn đề; không làm lại toàn bộ.

## URD/SRS Workflow

Khi user yêu cầu tạo URD, SRS, tài liệu đặc tả yêu cầu người dùng hoặc tài liệu dev/test ready:

1. Đọc `.claude/agents/urd-srs-agent.md`.
2. Đọc `.claude/skills/urd-srs-structure.md`.
3. Đọc các skill section-level bắt buộc khi viết URD/SRS:
   - `.claude/skills/urd-workflow-diagram.md` — II.1 Workflow Diagram
   - `.claude/skills/urd-function-tree.md` — II.2 Business Function Diagram
   - `.claude/skills/urd-permission-matrix.md` — II.3 Permission Matrix và II.4 RBAC Matrix
   - `.claude/skills/urd-sequence-diagram.md` — II.5 Sequence Diagram
   - `.claude/skills/urd-use-case.md` — III Use Case Specification
   - `.claude/skills/urd-screen-spec.md` — IV Giao diện chức năng
4. Nếu có `.claude/skills/urd-review-checklist.md`, đọc trước khi tự review hoặc gọi `ba-qa-agent`.
5. Nếu có template trong `.claude/input/urd-template.md`, dùng template đó làm tham chiếu format.
6. Phân tích Actors, chức năng chính, quy trình nghiệp vụ, phân quyền và thông tin còn thiếu.
7. Nếu thiếu thông tin critical, hỏi lại trước khi viết bản hoàn chỉnh.
8. Nếu không thiếu critical, viết tài liệu theo đúng cấu trúc:

```text
I.   Giới thiệu
II.  Các yêu cầu về tổng thể phần mềm
     II.1 Workflow Diagram
     II.2 Business Function Diagram
     II.3 Permission Matrix
     II.4 RBAC Matrix
     II.5 Sequence Diagram
III. Đặc tả tình huống sử dụng
IV.  Giao diện chức năng
C.   Yêu cầu phi chức năng
```

Đầu ra lưu tại:

```text
.claude/output/[tên_dự_án]/urd/urd-srs-v[N].md
```

Nếu thư mục dự án đang dùng pattern khác, giữ pattern hiện có và chỉ tăng version theo cách đang dùng.

Trong URD/SRS, được phép dùng `[Cần xác nhận: ...]` cho thông tin thiếu thật sự. Sau tài liệu phải có danh sách các mục cần xác nhận, assumptions và thông tin cần bổ sung từ PO/stakeholder.

Sau khi tạo hoặc refine URD/SRS:

- Tự động review bằng `ba-qa-agent`.
- Findings phải phân loại `CRITICAL`, `MAJOR`, `MINOR`.
- Nếu có `CRITICAL`, sửa đúng section có vấn đề và tăng version; không viết lại toàn bộ tài liệu.
- Chỉ chốt khi không còn `CRITICAL`.

## Wireframe, React Prototype Và Feedback

Khi user yêu cầu tạo hoặc refine wireframe:

- Đọc `.claude/agents/ba-wireframe-agent.md`.
- Đọc `.claude/skills/wireframe-design-system.md`.
- Đọc `.claude/skills/ui-feedback-triage.md`.
- Trước khi bắt đầu, kiểm tra design guideline riêng của dự án: `.claude/output/[tên_dự_án]/design-guideline.md` hoặc pattern hiện có tương đương trong thư mục output dự án. Nếu có, dùng file đó làm chuẩn chính; `wireframe-design-system.md` chỉ áp dụng cho phần không override.
- Bám sát solution/user flow đã có; nếu có User Stories và Acceptance Criteria thì trace về các artifact đó.
- Lưu version mới theo pattern `.claude/output/[tên_dự_án]/wireframe/wireframe-v[N].md`, hoặc theo pattern hiện có trong thư mục dự án.

Khi user yêu cầu tạo React prototype hoặc xem trực quan:

- Đọc `.claude/agents/ui-react-agent.md`.
- Đọc `.claude/skills/wireframe-design-system.md`.
- Đọc `.claude/skills/react-ui-generation.md`.
- Kiểm tra design guideline riêng của dự án trước khi áp dụng chuẩn chung.
- Tạo prototype dựa trên wireframe/solution đã chốt ở vòng hiện tại.
- Không biến prototype thành production code; prototype phục vụ review UI/UX và feedback nghiệp vụ.

Khi user đưa feedback giao diện:

- Đọc `.claude/agents/ui-feedback-agent.md`.
- Đọc `.claude/skills/ui-feedback-triage.md`.
- Triage feedback trước khi sửa: phân loại nội dung nào là lỗi, thay đổi layout, thay đổi copy, thay đổi luồng, hoặc yêu cầu mới.
- Chỉ sửa đúng phần đã được xác nhận; không tái tạo toàn bộ wireframe/prototype.

Khi user yêu cầu tạo prompt cho Lovable hoặc frontend prototype:

- Dùng wireframe version mới nhất làm nguồn chính.
- Nếu có `lovable-prompt-v[N].md` trong output dự án, refine từ version mới nhất thay vì viết lại từ đầu.
- UI label trong prompt phải ưu tiên tiếng Việt.
- Lưu version mới theo pattern `.claude/output/[tên_dự_án]/prototype/lovable-prompt-v[N].md`, hoặc theo pattern hiện có trong thư mục dự án.

## Ngôn Ngữ

Toàn bộ tài liệu BA phải viết bằng tiếng Việt có dấu đầy đủ.

Các ngoại lệ được phép dùng tiếng Anh:

- Tiêu đề Markdown khi phù hợp
- Thuật ngữ kỹ thuật như API, ERD, Gherkin, Backlog, Epic, Wireframe, URD, SRS, RBAC, Use Case
- Prompt frontend/Lovable có thể dùng tiếng Anh nếu nền tảng đích yêu cầu, nhưng UI label và nghiệp vụ phải giữ tiếng Việt khi phù hợp

Không viết tiếng Việt không dấu.
Không dùng từ tiếng Anh không phổ biến trong nội dung nghiệp vụ; nếu buộc phải dùng, giải thích hoặc dịch kèm.

## Chất Lượng Đầu Ra

Mọi đầu ra BA cuối cùng phải thực tế, nhất quán và có thể dùng để triển khai:

- Nêu rõ assumption.
- Highlight risks, unknowns và open questions.
- Giữ traceability rõ ràng. Nếu có backlog: `PRD/Input -> Epic -> Story -> Wireframe/Prototype -> URD/SRS`.
- Nếu không tạo backlog: `PRD/Input -> Solution -> Wireframe/Prototype -> URD/SRS`.
- Không bịa thông tin kỹ thuật khi thiếu input kỹ thuật.
- Không dùng `TBD`, `N/A` hoặc placeholder trong tài liệu hoàn chỉnh, trừ `[Cần xác nhận: ...]` trong URD/SRS.
- Chỉ số thành công phải đo lường được.
- Actor, role, permission và thuật ngữ nghiệp vụ phải nhất quán xuyên suốt.
- Với REVIEW, phát hiện lỗi trước; tóm tắt chỉ là phần phụ.
- URD/SRS phải đảm bảo traceability nội bộ: `Business Function -> Permission Matrix -> Use Case -> Giao diện`.

## Ghi Chú Riêng Cho Codex

Các file `.claude/agents/*.md` là định nghĩa prompt, không phải native Codex subagent. Hãy xem các file này như hướng dẫn vai trò BA/UI và áp dụng trong session Codex hiện tại. Chỉ chạy chúng như subagent độc lập khi user yêu cầu rõ hoặc khi task đủ lớn và user đã cho phép dùng subagent.

Nếu user nói "load BA agent", "use BA agent", "dùng BA agent", hoặc yêu cầu tạo/refine/review tài liệu BA, hãy nạp hành vi trong `AGENTS.md`, các rule liên quan và agent/skill tương ứng trước khi phản hồi.
