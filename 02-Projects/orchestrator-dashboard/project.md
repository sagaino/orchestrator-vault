---
title: "Orchestrator Dashboard"
type: project
project_id: orchestrator-dashboard
repository: "/Users/sagaino/ciniru/orchestrator-dashboard"
agent: agy
graphify: true
graphify_output: "/Users/sagaino/ciniru/orchestrator-dashboard/graphify-out/graph.json"
verification_defaults: ["typecheck", "lint", "build"]
blueprint: frontend-vite
template_version: 2
blueprint_policy_version: 3
template_checksum: "f7fba97b339bd2e06c2ba970a7d0413f105b23501cb1d24c978540de77299b2a"
scaffold_mode: DETERMINISTIC_TEMPLATE
tags: ["project", "frontend", "react", "vite"]
created: 2026-08-16
updated: 2026-08-16
sources: ["[[01-Knowledge/patterns/frontend/project-skeleton-template]]"]
---

# Orchestrator Dashboard

## Role in the Wiki

Project metadata used by Personal AI Orchestrator. Source code and Graphify output remain in the repository.

## Repository

- Repository: `/Users/sagaino/ciniru/orchestrator-dashboard`
- Coding agent: `agy`
- Graphify enabled: `true`
- Graphify output: `/Users/sagaino/ciniru/orchestrator-dashboard/graphify-out/graph.json`
- Verification defaults: `typecheck`, `lint`, `build`
- Scaffold mode: `DETERMINISTIC_TEMPLATE`
- Template version: `2`

## Task Queue

- Tasks: `02-Projects/orchestrator-dashboard/tasks/`

## Architecture Conventions

Coding agent **wajib** mengikuti konvensi struktur berikut agar konsistensi project terjaga:

### Struktur Halaman (Feature-Based Modular)

Setiap halaman/fitur menggunakan pola modular:

```
src/pages/<FeatureName>/
├── index.tsx              # Page entry, layout utama, komposisi komponen
├── hooks/
│   └── use<Feature>.ts    # Custom hooks (data fetching, state logic)
├── components/
│   ├── index.ts           # Barrel re-export semua komponen
│   └── <Component>.tsx    # Komponen UI spesifik halaman
└── types/
    └── <feature>.ts       # TypeScript interfaces & types
```

### State Management & Data Fetching

- Gunakan **TanStack Query hooks** dari `src/hooks/use-orchestrator.ts` untuk semua data fetching.
- Jangan gunakan `useState` + `useEffect` + manual fetch. Semua sudah tersedia sebagai hooks.
- SSE events dikelola **satu kali** di `src/providers/EventsProvider.tsx` — jangan subscribe ulang per halaman.

### Shared Components & Services

- Komponen shared global: `src/components/` (ErrorBoundary, layout, project)
- API client: `src/services/orchestrator.ts` (semua endpoint terpusat di `OrchestratorApi`)
- Konstanta auth: `src/lib/constant/auth.ts`
- Mutation hooks otomatis invalidasi cache via `queryKeys`.

### Styling

- Gunakan **Tailwind CSS** sesuai design system dark-theme project (`bg-slate-950`, `text-slate-100`, `border-slate-800`).
- Komponen UI primitif dari **Shadcn UI** (`src/components/ui/`), treat sebagai vendor — jangan edit langsung.

## Graphify Note

Graphify is refreshed in the repository by the orchestrator. Its output is never copied into the Vault.
