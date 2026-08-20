---
title: Frontend Architecture & Engineering Standards
type: concept
tags: [topic, frontend, architecture, standards, gallery-fmfu]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/AGENTS.md]]", "[[03-Sources/documentation/rules-react.md]]", "[[03-Sources/documentation/rules-api.md]]", "[[03-Sources/documentation/rules-workflow.md]]"]
---

# Topic Hub: Frontend Architecture & Engineering Standards

Pusat acuan arsitektur, aturan coding, dan alur kerja pengembangan Frontend untuk proyek **FMFU Photo Tagging Web & Gallery** (`gallery-fmfu`).

---

## 🏗 Architectural Concepts (`01-Knowledge/concepts/`)
- [[01-Knowledge/concepts/architecture/feature-driven-architecture|Feature-Driven Architecture]]: Struktur folder `src/pages/[Feature]/` dan aturan pembagian komponen global vs fitur.
- [[01-Knowledge/concepts/architecture/state-logic-separation|State & Logic Separation]]: Pemisahan state visual murni vs Custom Hook (`use[Feature].ts`) vs TanStack Query.
- [[01-Knowledge/concepts/architecture/api-service-data-flow|API Service Data Flow]]: Alur data wajib `UI → Custom Hook → Service → Axios → API`.
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]: Larangan keras (penulisan ulang berkas, `any` type, `localStorage` langsung, teks hardcoded).
- [[01-Knowledge/concepts/definition-of-done|Definition of Done (DoD)]]: Check-list verifikasi kodingan dan pemeriksaan tipe data via `npx tsc --noEmit`.

---

## 🛠 Stack & Entities (`01-Knowledge/concepts/`)
- [[01-Knowledge/concepts/react/react|React & Component Rules]]: Aturan komponen presentation murni, batas baris berkas (<300 baris), dan 4 UI State.
- [[01-Knowledge/concepts/state-management/tanstack-query|TanStack Query]]: Single source of truth untuk server state dan manajemen cache.
- [[01-Knowledge/concepts/api/axios-client|Axios Client & Services]]: Pengelolaan HTTP request gateway, interceptor, dan pembungkusan error.
- [[01-Knowledge/concepts/react/react-hook-form|React Hook Form & Zod]]: Standar formulir bertipe statis dan validasi skema.
- [[01-Knowledge/concepts/react/i18next|i18next Localization]]: Pengelolaan lokalisasi bahasa (default `id`).

---

## 📋 Comprehensive Synthesis (`01-Knowledge/patterns/frontend/`)
- [[01-Knowledge/patterns/frontend/fe-development-guidelines|FE Development Guidelines]]: Panduan lengkap ringkasan aturan AI & Frontend engineering.
