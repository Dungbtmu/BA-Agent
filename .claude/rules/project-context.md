# Project Context — Bối cảnh hệ thống

## Mục tiêu

Repo này là **BA Agent System** — bộ công cụ hỗ trợ BA tạo tài liệu phân tích nghiệp vụ cho **mọi dự án, mọi domain** (Fintech, SaaS, EdTech, E-commerce, Logistics, AI...).

Loại tài liệu đầu ra đa dạng, tùy theo yêu cầu: Backlog, Epic, User Story, Wireframe, URD (User Requirements Document), SRS (Software Requirements Specification).

Đây là hệ thống dựa trên tài liệu Markdown — không có code để build, test, hay compile.

## Phạm vi BA Agent

- **Tập trung**: Nghiệp vụ, user flow, requirement, acceptance criteria, wireframe, React UI prototype (để feedback), URD/SRS dev/test-ready
- **KHÔNG xử lý**: Kiến trúc kỹ thuật, ERD, API spec chi tiết, infrastructure, production code (đó là phạm vi của SA/Dev)

## Team và vai trò tham chiếu

| Vai trò | Trách nhiệm |
|---|---|
| **PO** | Yêu cầu nghiệp vụ (scope theo từng giai đoạn) — có thể là PRD, ghi chú, mô tả miệng, hoặc tài liệu bất kỳ |
| **BA** _(repo này)_ | Backlog, Epic, User Story, Wireframe, URD, SRS — tùy loại tài liệu yêu cầu; đầu vào đa dạng, không quy định |
| **Dev** | Triển khai code dựa trên tài liệu PO + BA |
| **Tester** | Kiểm thử dựa trên tài liệu và sản phẩm |

## Cấu trúc thư mục

```
.claude/
├── agents/                    ← Định nghĩa các subagent BA chuyên biệt
│   ├── ba-research-agent.md
│   ├── ba-clarification-agent.md
│   ├── ba-solution-agent.md
│   ├── ba-devil-advocate-agent.md
│   ├── ba-backlog-agent.md
│   ├── ba-wireframe-agent.md
│   ├── ui-react-agent.md
│   ├── ui-feedback-agent.md
│   ├── urd-srs-agent.md
│   ├── ba-qa-agent.md
│   ├── ba-postcheck-agent.md
│   ├── ba-process-summary-agent.md
│   └── ba-agent-fix-agent.md  ← Apply review report từ Agent Review Agent
├── rules/                     ← Quy tắc bắt buộc, load mọi session
│   ├── project-context.md
│   ├── ba-persona.md
│   ├── language.md
│   ├── agent-workflow.md      ← pipeline + intent recognition + chế độ vận hành
│   ├── output-schema.md
│   └── review-handoff-policy.md  ← Quy tắc phạm vi sửa khi apply review report
├── skills/                    ← Skill chuyên biệt, agent đọc khi cần — tổ chức theo nhóm
│   ├── clarification/         ← Phase 1: làm rõ yêu cầu
│   │   ├── requirement-clarification.md  ← orchestrator
│   │   ├── input-analysis.md
│   │   ├── as-is-analysis.md
│   │   ├── domain-research.md
│   │   ├── domain-gap-analysis.md
│   │   └── problem-framing.md
│   ├── solution/              ← Phase 1: thiết kế giải pháp
│   │   ├── stakeholder-mapping.md
│   │   ├── user-persona-identification.md
│   │   ├── context-constraint-analysis.md
│   │   ├── assumption-risk-analysis.md
│   │   └── solution-critique.md
│   ├── ui/                    ← Phase 2: giao diện
│   │   ├── wireframe-design-system.md
│   │   ├── react-ui-generation.md
│   │   └── ui-feedback-triage.md
│   ├── urd/                   ← Phase 3: tài liệu URD/SRS
│   │   ├── urd-srs-structure.md  ← orchestrator
│   │   ├── urd-workflow-diagram.md
│   │   ├── urd-function-tree.md
│   │   ├── urd-permission-matrix.md
│   │   ├── urd-sequence-diagram.md
│   │   ├── urd-use-case.md
│   │   ├── urd-screen-spec.md
│   │   ├── urd-review-checklist.md
│   │   └── document-integrity-check.md
│   ├── sync/                  ← SYNC mode: đồng bộ artifact khi requirement thay đổi
│   │   ├── change-handler.md
│   │   ├── impact-analysis.md
│   │   ├── artifact-patch.md
│   │   └── traceability-map.md
│   ├── shared/                ← Dùng chung xuyên phase
│   │   ├── resolve-oqs.md
│   │   ├── process-log.md
│   │   └── review-report-fix-handoff.md
│   └── diagram/               ← Skill vẽ diagram (Excalidraw) — cấu trúc riêng
├── input/                     ← Tài liệu đầu vào từ PO/stakeholder
│   └── review-reports/        ← Inbox nhận review report từ Agent Review Agent
│       └── [tên-report].md
└── output/                    ← Tài liệu BA tạo ra, tổ chức theo dự án
    └── [tên_dự_án]/
        ├── research/          ← Output từ ba-research-agent và domain-gap-analysis
        │   ├── domain-brief.md
        │   └── domain-gap-analysis.md   ← (nếu có Domain Brief + client input)
        ├── wireframe/
        ├── urd/
        ├── solution/
        └── process-summary.md
```

## Review/Fix Bridge

Hệ thống có bridge để nhận và apply review report từ Agent Review Agent:

- **Inbox report:** `.claude/input/review-reports/[tên-report].md`
- **Agent xử lý:** `ba-agent-fix-agent` — đọc report, convert thành patch plan, sửa đúng file
- **Policy:** `.claude/rules/review-handoff-policy.md` — quy tắc allowlist file, điều kiện ASK_CONFIRM
- **Skill parse:** `.claude/skills/shared/review-report-fix-handoff.md` — schema finding, xử lý từng Action Type
- **Output:** `.claude/input/review-reports/fix-summary-[tên-report].md` — báo cáo kết quả sau khi apply
