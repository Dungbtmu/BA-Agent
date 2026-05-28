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
│   ├── ba-clarification-agent.md
│   ├── ba-solution-agent.md
│   ├── ba-backlog-agent.md
│   ├── ba-wireframe-agent.md
│   ├── ba-qa-agent.md
│   ├── ui-react-agent.md
│   ├── ui-feedback-agent.md
│   └── urd-srs-agent.md
├── rules/                     ← Quy tắc bắt buộc, load mọi session
│   ├── project-context.md
│   ├── ba-persona.md
│   ├── language.md
│   ├── ba-workflow.md
│   ├── agent-workflow.md
│   └── output-schema.md
├── skills/                    ← Skill chuyên biệt, agent đọc khi cần
│   ├── [BA skills]            ← problem-framing, requirement-clarification, stakeholder-mapping...
│   ├── [UI skills]            ← react-ui-generation, ui-feedback-triage, wireframe-iteration
│   └── [URD skills]           ← urd-srs-structure (orchestrator) + 6 skill con theo từng section
├── input/                     ← Tài liệu đầu vào từ PO/stakeholder
└── output/                    ← Tài liệu BA tạo ra, tổ chức theo dự án
    └── [tên_dự_án]/
        ├── wireframe/
        ├── urd/
        └── solution/
```
