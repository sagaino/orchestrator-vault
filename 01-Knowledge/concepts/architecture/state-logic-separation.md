---
title: State & Logic Separation
type: concept
tags: [concept, react, hooks, state-management]
created: 2026-08-12
updated: 2026-08-12
sources: ["[[03-Sources/documentation/AGENTS.md]]", "[[03-Sources/documentation/rules-react.md]]"]
---

# State & Logic Separation

Prinsip pemisahan antara tampilan UI (*presentational view*) dengan logika bisnis dan efek samping (*side-effects*).

## Hirarki & Aturan
1. **Simple Local UI State**:
   - Hanya untuk toggle visual murni tanpa `useEffect` atau panggilan API (contoh: `isOpen`, `activeTab`).
   - Boleh ditaruh langsung di komponen `.tsx`.
2. **Custom Hook Separation (`use[Feature].ts`)**:
   - Wajib diekstrak ke Custom Hook jika mengandung: `useEffect` (misal listener event keyboard/window), interaksi API, progres unduhan, atau logika multi-state kompleks.
   - Komponen `.tsx` hanya mengonsumsi return value dari hook tersebut.
3. **TanStack Query**:
   - Digunakan sebagai *single source of truth* untuk seluruh server state.
   - Wajib melakukan invalidate atau pembaruan cache query setelah mutasi berhasil.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/state-management/tanstack-query|TanStack Query]]
