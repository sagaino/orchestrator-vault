---
supersession: "ACTIVE"
review_by: "2027-02-18"
owner: "knowledge-curator"
confidence: 0.5
orchestrator_run: "legacy-migration:p8-0-provenance-20260822T103839Z-efa8f1a9"
provenance_schema: 1
title: Summary of AI Engineering Guide (AGENTS.md)
type: concept
tags: [summary, agents, architecture, gallery-fmfu]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/AGENTS.md]]"]
---

# Summary: AI Engineering Guide (AGENTS.md)

## Key Takeaways
- **Tech Stack**: React (TypeScript), Vite, Tailwind CSS, shadcn/ui, TanStack Query, React Router, i18next.
- **Rule Hierarchy**: User prompt > `AGENTS.md` & `.agents/rules/` > Existing architecture > Framework best practices.
- **Forbidden Behaviors**: No full-file rewrites, no direct `localStorage` access for auth (use `useLocalStorage`), no hardcoded user strings (use `i18next`), no `any` types, no temporary debug code (`console.log`).
- **Feature-Driven Layout**:
  - Global shared UI in `src/components/ui/`
  - Feature-encapsulated code in `src/pages/[Feature]/` (components, hooks, types)
  - Global services in `src/services/`
  - Core utils in `src/lib/`

## Related Pages
- [[01-Knowledge/concepts/architecture/feature-driven-architecture|Feature-Driven Architecture]]
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
