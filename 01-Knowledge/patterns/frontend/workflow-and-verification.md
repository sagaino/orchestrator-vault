---
title: Summary of Workflow & AI Behavior Rules
type: pattern
tags: [summary, workflow, verification, dod]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-workflow.md]]"]
---

# Summary: Workflow & AI Behavior Rules

## Key Takeaways
- **Task Classification**: Classify tasks as New Feature, Bug Fix, Refactor, or UI Improvement before writing code.
- **Verification Protocol**: Always run `npx tsc --noEmit` after code edits. Run `npm run build` when modifying configs/dependencies.
- **Editing Policy**: Minimal diffs, preserve naming and comments. No unrelated refactorings.
- **Definition of Done (DoD)**: Type checking passed, Feature-Driven structure respected, 4 UI states handled, `i18next` localized.

## Related Pages
- [[01-Knowledge/concepts/definition-of-done|Definition of Done (DoD)]]
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
