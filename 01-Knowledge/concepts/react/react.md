---
title: React & Component Rules
type: concept
tags: [entity, react, components, size-limits]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-react.md]]"]
---

# React & Component Rules

Aturan pembuatan komponen UI dan pembatasan ukuran berkas pada proyek FE.

## Arsitektur Komponen
- **Global UI (`src/components/ui/`)**: Komponen presentational murni berbasis `shadcn/ui`. Bebas dari API, TanStack Query, routing, atau logika bisnis.
- **Feature Component (`src/pages/[Feature]/components/`)**: Komponen tampilan khusus fitur yang menerima data dari Custom Hook.

## Batas Ukuran Berkas (File Size Limits)
- **Component**: < 300 baris
- **Hook**: < 250 baris
- **Service**: < 200 baris
- **Utility**: < 150 baris

## 4 UI States Wajib
Setiap layar data wajib menangani 4 kondisi:
1. **Loading**: Tampilan Skeleton loader.
2. **Success**: Tampilan data sukses.
3. **Empty**: Tampilan pesan saat data kosong.
4. **Error**: Tampilan error beserta tombol retry (ulang).

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/architecture/state-logic-separation|State & Logic Separation]]
