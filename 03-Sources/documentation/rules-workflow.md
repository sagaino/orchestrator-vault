---
trigger: always_on
description: AI task classification, editing policy, verification, and Definition of Done.
---

# Workflow & AI Behavior Rules

## Editing & Refactoring Policy
- **Editing**: Small, isolated edits with minimal diffs. Preserve formatting, comments, and naming.
- **Refactoring**: Unrelated refactoring prohibited. Minor scoped cleanup within actively edited files is encouraged to prevent technical debt.
- **Dependencies**: Reuse existing utils/libraries before introducing new packages (requires user approval).
- **File Creation**: Reuse or extend existing files first. Avoid parallel architectures.

## Task Workflows
Classify task into exactly one type before coding:
- **New Feature**: Types → Endpoints → Service → Hook → Component → Page → Routes.
- **Bug Fix**: Understand root cause → Smallest targeted fix → Verify.
- **Refactor**: Improve structure without changing behavior. Confirm necessity first.
- **UI Improvement**: Preserve business logic. Follow Figma/design system.

## Verification & Confidence
- **Type Check**: Always run `npx tsc --noEmit` after code changes.
- **Build Check**: Run `npm run build` only when changing config, routing, dependencies, or Vite build.
- **Confidence Levels**:
  - High: Proceed.
  - Medium: State assumptions clearly.
  - Low: Stop and ask user.

## Definition of Done (DoD) Checklist
- ✓ Feature-Driven structure respected; shared modules reused.
- ✓ Strongly typed (`no any`), readable, maintainable, minimal diffs.
- ✓ Business logic in hooks; UI is presentational; React Query cache managed.
- ✓ Loading/Error/Empty states handled; responsive; accessible; `i18next` applied.
- ✓ Scope respected; security preserved; type checked via `npx tsc --noEmit`.
