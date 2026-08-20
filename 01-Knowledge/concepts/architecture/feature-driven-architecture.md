---
title: Feature-Driven Architecture
type: concept
tags: [concept, architecture, feature-driven, layout]
created: 2026-08-12
updated: 2026-08-12
sources: ["[[03-Sources/documentation/AGENTS.md]]"]
---

# Feature-Driven Architecture

Aturan struktur folder dan enkapsulasi fitur pada proyek FE.

## Folder Layout (`src/`)

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

## Aturan Utama
1. **Feature Encapsulation**: Semua komponen, hook, dan tipe data khusus fitur wajib disimpan di dalam folder fitur `src/pages/[Feature]/`.
2. **Shared Modules**: Komponen yang digunakan oleh lebih dari satu fitur berada di `src/components/ui/`, `src/hooks/`, atau `src/lib/`.
3. **Path Aliases**: Selalu gunakan alias `@/` yang mengarah ke `src/`. Dilarang menggunakan relative import panjang seperti `../../../`.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/architecture/state-logic-separation|State & Logic Separation]]
