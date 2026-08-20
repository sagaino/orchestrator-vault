# AGENTS.md — AI Engineering Guide

Project: **FMFU Photo Tagging Web & Gallery** (`gallery-fmfu`)

Stack:
- React (with TypeScript)
- Vite
- Tailwind CSS
- shadcn/ui
- TanStack Query
- React Router
- i18next

---

# Execution & Operating Principles

Always follow this priority when rules conflict:
1. Explicit user instructions in current prompt
2. `AGENTS.md` & companion rules in `.agents/rules/`
3. Existing project architecture
4. Framework best practices
5. Default AI behavior

Core Operating Workflow:
- **Search before creating**: Reuse existing components, hooks, utilities, and patterns before adding new ones.
- **Context-aware decisions**: Inspect existing code conventions first. Ask the user only when requirements or business logic remain ambiguous.
- **Focused scope**: Keep diffs minimal and reviewable. Small refactorings or cleanups within actively edited files are allowed to prevent technical debt, but avoid unnecessary rewrites of unrelated files.
- **Verify before finishing**: Validate changes with `npx tsc --noEmit` and relevant checks.

---

# Forbidden Behaviors

Never:
- Rewrite an entire file when only a localized edit is needed.
- Refactor unrelated components, pages, or modules.
- Introduce new npm dependencies without user approval.
- Invent API contracts, request payloads, response models, or business rules.
- Bypass the Service Layer or Axios client (UI → Hook → Service → Axios → API).
- Access `localStorage` directly for authentication data (use central `useLocalStorage`).
- Hardcode user-facing strings instead of using `i18next` (`useTranslation()`).
- Leave temporary debug code (`console.log`, `TODO`, `FIXME`) in production code.
- Rename exported APIs, components, or files unnecessarily.

---

# Architecture & Standards

Feature-Driven Architecture mandatory layout:

```
src/
├── components/ui/       # Shared presentation (shadcn/ui, atomic UI)
├── hooks/               # Shared non-business React hooks
├── lib/                 # Core utils (axios, i18n, error-utils, types)
├── services/            # Global API service layer
├── pages/               # Feature-based pages
│   └── [Feature]/       # Feature module: components/, hooks/, types/, index.tsx
└── routes/              # Routing configuration (PrivateRoute / PublicRoute)
```

Rules:
- **Feature Encapsulation**: Keep feature-specific components, hooks, and types inside their feature directory under `src/pages/[Feature]/`. Shared modules stay in global directories (`src/components/ui`, `src/hooks`, `src/lib`, `src/services`).
- **Path Aliases**: Always use `@/` alias for `src/` imports. Never use deep relative imports (`../../../`).
- **Type Safety**: Explicit TypeScript types everywhere. Avoid `any`.
- **Localization**: Default locale `id`. Add missing keys in `src/lib/i18n/locales/` and use `useTranslation()`.
- **Routing**: Use `PrivateRoute` for protected pages and `PublicRoute` for public pages.
- **State Management & Logic Separation**: TanStack Query for server state. Pure local UI presentation state (e.g. simple `isOpen`, `activeTab` without `useEffect`/API) may stay in component `.tsx`. Any state with side-effects (`useEffect`), API interactions, keyboard listeners, or complex logic **must be extracted** into a Custom Hook (`use[Feature].ts`).
