# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Repo này là **BA Agent System** — hỗ trợ BA tạo tài liệu phân tích nghiệp vụ (Backlog, Epic, User Story, Wireframe, React UI prototype, URD/SRS) cho mọi dự án, mọi domain. Không có code production để build, test, hay compile.

BA agent tập trung vào **nghiệp vụ và giao diện mức BA** — bao gồm cả gen React prototype để stakeholder feedback — không xử lý kiến trúc kỹ thuật hay production code.

## Rules chi tiết

Các quy tắc được tách thành file riêng trong `.claude/rules/`:

- [`project-context.md`](.claude/rules/project-context.md) — Bối cảnh hệ thống, phạm vi BA, team, cấu trúc
- [`ba-persona.md`](.claude/rules/ba-persona.md) — Phong cách, tư duy, năng lực BA Senior
- [`language.md`](.claude/rules/language.md) — Quy tắc ngôn ngữ (tiếng Việt có dấu bắt buộc)
- [`ba-workflow.md`](.claude/rules/ba-workflow.md) — Quy trình GENERATE / REFINE / REVIEW
- [`agent-workflow.md`](.claude/rules/agent-workflow.md) — Quy trình phối hợp các subagent BA
- [`output-schema.md`](.claude/rules/output-schema.md) — Tiêu chuẩn định dạng Epic, Story, Wireframe

