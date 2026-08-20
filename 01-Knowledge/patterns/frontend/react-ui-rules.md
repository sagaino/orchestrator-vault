---
title: Summary of React & UI Rules
type: pattern
tags: [summary, react, ui, hooks, forms]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-react.md]]"]
---

# Summary: React & UI Rules

## Key Takeaways
- **File Size Limits**: Component < 300 lines | Hook < 250 lines | Service < 200 lines | Utility < 150 lines.
- **State & Logic Separation**:
  - Simple visual toggles (`isOpen`, `activeTab`) may remain in `.tsx`.
  - Any state with `useEffect`, API calls, or complex multi-state logic **must be extracted to a custom hook** (`use[Feature].ts`).
- **Forms & Validation**: `react-hook-form` + `zod` via `zodResolver`. Strongly typed `useForm<z.infer<typeof schema>>()`.
- **4 Mandatory UI States**: Loading (Skeletons), Success, Empty, and Error (with retry).
- **Localization**: No hardcoded text; always use `useTranslation()`.

## Related Pages
- [[01-Knowledge/concepts/architecture/state-logic-separation|State & Logic Separation]]
- [[01-Knowledge/concepts/react/react-hook-form|React Hook Form & Zod]]
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
