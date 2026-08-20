---
title: FE Development Guidelines
type: pattern
tags:
  - synthesis
  - fe
  - guidelines
  - standards
created: 2026-08-12
updated: 2026-08-14
sources:
  - "[[03-Sources/documentation/AGENTS.md]]"
  - "[[03-Sources/documentation/rules-react.md]]"
  - "[[03-Sources/documentation/rules-api.md]]"
  - "[[03-Sources/documentation/rules-workflow.md]]"
---

# FE Development Guidelines

Panduan komprehensif alur kerja, standar arsitektur, dan aturan koding untuk proyek **FMFU Photo Tagging Web & Gallery**.

---

## 1. Arsitektur Berbasis Fitur (Feature-Driven Architecture)
Setiap fitur dienkapsulasi secara terisolasi di dalam `src/pages/[Feature]/`:
- `components/`: Komponen visual spesifik fitur.
- `hooks/`: Custom hook khusus fitur (`use[Feature].ts`).
- `types/`: Skema Zod dan tipe data khusus fitur.
- `index.tsx`: Halaman utama fitur.

Modul bersama (shared) bertempat di `src/components/ui/`, `src/hooks/`, `src/lib/`, dan `src/services/`. Gunakan selalu import path alias `@/`.

---

## 2. Pemisahan Logika & State Management
- **Visual Toggles**: State UI sederhana tanpa `useEffect`/API (misal `isOpen`) boleh berada di `.tsx`.
- **Custom Hook (`use[Feature].ts`)**: Wajib digunakan untuk efek samping (`useEffect`), keyboard listeners, atau interaksi API.
- **TanStack Query**: Digunakan untuk mengelola server state & caching.

---

## 3. Komunikasi API
- **Alur Mandatori**: `UI Component → Custom Hook → Service Layer → Axios Client → Backend API`.
- Seluruh endpoint di-sentralisasi di `src/lib/constant/endpoints.ts`.
- Penanganan error melalui `src/lib/error-utils.ts`.

---

## 4. Batas Ukuran Berkas & 4 UI States
- **Ukuran Berkas**: Komponen < 300 baris | Hook < 250 baris | Service < 200 baris | Utility < 150 baris.
- **4 UI States**: Loading (Skeletons), Success, Empty, dan Error (dengan opsi retry).

---

## 5. Protokol Verifikasi
Selalu jalankan `npx tsc --noEmit` setelah melakukan perubahan kode untuk memastikan tidak ada kesalahan tipe data TypeScript.

---

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]
- [[01-Knowledge/concepts/definition-of-done|Definition of Done (DoD)]]
