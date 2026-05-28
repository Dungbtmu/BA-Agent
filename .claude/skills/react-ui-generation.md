---
name: react_ui_generation
description: Quy tắc gen React component từ wireframe BA — đủ để render trực quan, không chứa business logic thật
tools: []
---

# Skill: React UI Generation

## Mục tiêu

Chuyển wireframe text-based của BA thành React component có thể chạy được — đủ để stakeholder nhìn thấy giao diện thực tế và đưa feedback, không phải để production.

## Input

- `wireframe`: nội dung wireframe text-based (component list, layout, interactions, states)
- `design_system`: optional — tên thư viện UI đang dùng (Tailwind, shadcn/ui, MUI, Ant Design...) — nếu không có, xem `output/[tên_dự_án]/design-guideline.md` hoặc dùng chuẩn từ `wireframe-design-system.md`
- `existing_components`: optional — danh sách component đã có để tái sử dụng

## Output

- React component(s) cho từng màn hình trong wireframe
- Mock data tối thiểu để UI có thể render
- Ghi chú rõ phần nào là placeholder cần thay bằng logic thật

## Thinking Pattern

1. **Màn hình nào?** — Mỗi màn hình trong wireframe = 1 component hoặc 1 page
2. **Component nào tái sử dụng được?** — Button, Input, Modal... dùng từ design system thay vì tự viết
3. **State nào cần thiết để UI trông đúng?** — Chỉ UI state (open/close, loading, error display), không phải business state
4. **Mock data trông như thế nào?** — Đủ realistic để stakeholder đánh giá layout
5. **Phần nào là placeholder?** — API call, auth, business logic → comment rõ `// TODO: replace with real logic`

## Execution

**Bước 1 — Đọc wireframe, xác định danh sách component**
- Mỗi màn hình → 1 page component
- Các block lặp lại (card, row, item) → tách thành sub-component
- Xác định props cần thiết cho từng component

**Bước 2 — Xác định UI states cần demo**
- Default state (dữ liệu bình thường)
- Empty state (chưa có dữ liệu)
- Loading state (nếu có async action trong wireframe)
- Error state (nếu wireframe có error handling)

**Bước 3 — Gen component**
- Dùng functional component + TypeScript
- Props có type rõ ràng
- Mock data đặt ở đầu file hoặc file riêng `mockData.ts`
- Không gọi API thật — dùng mock hoặc `// TODO`
- Không implement business logic — dùng `console.log` hoặc `// TODO` cho handlers

**Bước 4 — Ghi chú placeholder**
- Mỗi `// TODO` phải ghi rõ: cần thay bằng gì, ví dụ:
  `// TODO: replace with useQuery('/api/campaigns')`
  `// TODO: implement form validation`

## Rules

- **Không implement business logic** — handlers chỉ cần đủ để UI response (open modal, show/hide element)
- **Không gọi API thật** — luôn dùng mock data
- **Không tự thêm tính năng** không có trong wireframe
- Dùng component từ design system khi có — không tự viết từ đầu
- File structure theo màn hình, không theo feature (vì đây là prototype)
- Nếu wireframe không rõ một component → hỏi trước khi tự suy diễn

## Output Format

````
## React Components — [Tên màn hình]

**Wireframe liên kết:** WF-[ID]
**Design system:** [Tailwind + shadcn/ui / MUI / ...]

---

### [ComponentName].tsx

```tsx
// [Mô tả ngắn: màn hình này làm gì]
// Wireframe: WF-[ID]

import ...

// Mock data — replace with real API call
const mockData = { ... }

type [ComponentName]Props = {
  ...
}

export function [ComponentName]({ ... }: [ComponentName]Props) {
  // UI state only
  const [isOpen, setIsOpen] = useState(false)

  return (
    // JSX theo layout trong wireframe
  )
}
```

### Placeholder cần thay khi dev thật

| TODO | Vị trí | Cần thay bằng |
|---|---|---|
| Mock data | `mockData.ts` | `useQuery('/api/...')` |
| Submit handler | `handleSubmit` | Form validation + API call |
````

## Failure Cases

- Gen component có gọi API thật → block khi chạy prototype
- Tự thêm tính năng không có trong wireframe → stakeholder feedback lệch
- Không ghi `// TODO` → dev sau không biết phần nào là placeholder
- Tạo quá nhiều abstraction (hooks, context, store) cho prototype → khó sửa khi feedback
- Không có mock data realistic → stakeholder không đánh giá được layout thực tế
