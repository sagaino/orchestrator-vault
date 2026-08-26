---
supersession: "ACTIVE"
review_by: "2027-02-18"
owner: "knowledge-curator"
confidence: 0.5
orchestrator_run: "legacy-migration:p8-0-provenance-20260822T103839Z-efa8f1a9"
provenance_schema: 1
title: React Hook Form & Zod
type: concept
tags: [entity, forms, validation, zod, react-hook-form]
created: 2026-08-12
updated: 2026-08-14
sources: ["[[03-Sources/documentation/rules-react.md]]"]
---

# React Hook Form & Zod

Standar pengelolaan formulir dan validasi skema pada aplikasi.

## Aturan Penggunaan
- **Pustaka**: `react-hook-form` terintegrasi dengan `zod` melalui `zodResolver`.
- **Lokasi Skema**: Skema validasi khusus fitur di `src/pages/[Feature]/types/`, skema umum di `src/lib/types/`.
- **Tipe Statis**: Deklarasi form wajib bertipe statis: `useForm<z.infer<typeof schema>>()`.
- **UX Form**: Nonaktifkan tombol kirim saat status `isSubmitting` bernilai `true` dan tampilkan error validasi dari `formState.errors`.

## Halaman Terkait
- [[01-Knowledge/concepts/architecture/frontend-architecture|Frontend Architecture Hub]]
- [[01-Knowledge/concepts/react/react|React & Component Rules]]
