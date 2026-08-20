---
trigger:
  includes: ["src/components/**", "src/pages/**/*.tsx", "src/hooks/**"]
description: Rules for React components, hooks, state management, forms, and UI.
---

# React & UI Rules

## Component Architecture
- **Global UI (`src/components/ui/`)**: Pure presentational. No API, business logic, React Query, or routing.
- **Feature Component (`src/pages/[Feature]/components/`)**: Feature rendering. Logic lives in custom hooks.
- **File Size Limits**: Component < 300 lines | Hook < 250 lines | Service < 200 lines | Utility < 150 lines.

## State & Business Logic
- **State Priority**: Local React State → Custom Hook → TanStack Query (server state).
- **State & Logic Separation**:
  - **Simple Local UI State**: Pure visual toggles without `useEffect` or API calls (e.g. `isOpen`, `activeTab`) can stay directly in component `.tsx`.
  - **Custom Hooks Separation**: Any state involving `useEffect` (e.g. keyboard navigation, window event listeners), API calls/download progress, or complex multi-state logic **must be extracted** to a custom hook (`use[Feature].ts`). Views consume hook outputs.
- **useEffect**: External system synchronization only. Never derive state or replace event handlers with `useEffect`.
- **TanStack Query**: Single source of truth for server state. Invalidate or update cache on successful mutation.
- **Performance**: Use `useMemo`, `useCallback`, `React.memo` only when measurable benefits exist.

## Forms & Validation
- **Engine**: `react-hook-form` + `zod` via `zodResolver`.
- **Schemas**: Feature schemas in `src/pages/[Feature]/types/`, shared schemas in `src/lib/types/`.
- **Controls**: Strongly typed `useForm<z.infer<typeof schema>>()`. Disable submit button during `isSubmitting`. Show `formState.errors`.

## TypeScript & Imports
- **Type Safety**: Explicit types everywhere (`no any`). Models in `src/lib/types/` or `src/pages/[Feature]/types/`.
- **Import Order**: 1. React → 2. Packages → 3. `@/` Aliases → 4. Relative → 5. Types.

## UX, Accessibility & Error Handling
- **4 UI States**: Every data screen requires Loading (Skeletons), Success, Empty, and Error (with retry).
- **Error Boundaries**: Wrap major UI sections/routes to prevent blank screens with fallback UI.
- **Accessibility**: Semantic HTML (`<button>` over `<div>`), keyboard nav, visible focus, ARIA labels.
- **Responsiveness & Layout**: Responsive layouts (mobile/tablet/desktop). Truncate long dynamic text with `title` tooltip.
- **Design System**: Reuse `shadcn/ui` from `@/components/ui/`. Hardcoded text forbidden (`useTranslation()`).
