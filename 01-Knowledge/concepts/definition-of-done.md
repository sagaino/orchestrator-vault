---
title: Definition of Done (DoD) & Verification
type: concept
tags: [concept, workflow, dod, verification]
created: 2026-08-12
updated: 2026-08-12
sources: ["[[03-Sources/documentation/rules-workflow.md]]"]
---

# Definition of Done (DoD) & Verification

Kriteria penyelesaian tugas dan protokol verifikasi sebelum menyatakan sebuah pekerjaan selesai.

## Protokol Verifikasi Kode
- **Type Check Wajib**: Selalu jalankan `npx tsc --noEmit` setelah melakukan perubahan kode TypeScript.
- **Build Check**: Jalankan `npm run build` jika terjadi perubahan pada konfigurasi, routing, atau dependency Vite.

## Checklist Definition of Done (DoD)
- [ ] **Struktur**: Mengikuti Feature-Driven structure (`src/pages/[Feature]/`) & menggunakan modul shared.
- [ ] **Type Safety**: Tipe data eksplisit (`no any`), mudah dibaca, dan diff minimal.
- [ ] **Pemisahan Logika**: Logika bisnis berada di Custom Hook; UI bersifat presentational; cache TanStack Query terkelola dengan benar.
- [ ] **4 UI States**: Menangani kondisi Loading (Skeletons), Success, Empty, dan Error (dengan opsi retry).
- [ ] **Aksesibilitas & Lokalisasi**: Responsif, terstruktur secara akseseibel (ARIA/keyboard), dan seluruh teks menggunakan `i18next`.
- [ ] **Verifikasi**: Tidak ada pergeseran scope, aspek keamanan terjaga, serta lulus type check `npx tsc --noEmit`.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/forbidden-behaviors|Forbidden Behaviors]]
