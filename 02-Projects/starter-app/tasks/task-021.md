---
title: "Implementasi Fitur dan Slicing UI Halaman Upload Image (/upload-img)"
type: task
task_id: FE-021
project: starter-app
status: DONE
tags: [task, starter-app, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["typecheck", "build"]
allowed_paths: ["src/lib/constant/routes.ts", "src/routes/index.tsx", "src/pages/UploadImg/index.tsx", "src/pages/UploadImg/components/UploadHeader.tsx", "src/pages/UploadImg/components/UploadHero.tsx", "src/pages/UploadImg/components/UploadDropzone.tsx", "src/pages/UploadImg/components/UploadFooter.tsx"]
requires_changes: true
risk: LOW
sources: ["03-Sources/assets/ui-mockups/2026-08-19-ui-upload-c2ca53.png"]
---

# Implementasi Fitur dan Slicing UI Halaman Upload Image (/upload-img)

## Permintaan User

buatkan feature baru dengan path upload-img. dan slicing feature ini sesuai dengan desain dari img yang saya berikan. untuk logo image bagian footer sudah tersedia di folder assets/

## UI Mockup References

- ![[03-Sources/assets/ui-mockups/2026-08-19-ui-upload-c2ca53.png]]
  - Path: `03-Sources/assets/ui-mockups/2026-08-19-ui-upload-c2ca53.png`

## Tujuan

Membuat dan mengimplementasikan fitur halaman baru pada path /upload-img dengan melakukan slicing UI presisi sesuai referensi mockup yang terlampir serta memanfaatkan aset logo yang sudah tersedia di folder assets.

## Scope

- `src/lib/constant/routes.ts`
- `src/routes/index.tsx`
- `src/pages/UploadImg/index.tsx`
- `src/pages/UploadImg/components/UploadHeader.tsx`
- `src/pages/UploadImg/components/UploadHero.tsx`
- `src/pages/UploadImg/components/UploadDropzone.tsx`
- `src/pages/UploadImg/components/UploadFooter.tsx`

## Hasil Yang Diharapkan

Fitur baru halaman upload image (/upload-img) berhasil dibuat dan dislicing presisi sesuai referensi mockup, terintegrasi ke router aplikasi, dan lolos verifikasi build serta typecheck.

## Acceptance Criteria

1. Halaman baru tersedia dan dapat diakses pada route /upload-img
2. Layout dan styling halaman Upload Img sesuai dengan visual mockup (Header navigasi Langkah Rasa, Hero/Info section di kiri dengan badge format file, Drag & Drop Upload card di kanan dengan tombol CHOOSE PHOTOS, serta Footer sponsor & partner resmi)
3. Assets logo (langkah_rasa_img, nasi_gandul_img, cuit_img, lookin_img, tix_fly_img) terpasang dengan rapi di header dan footer sesuai mockup
4. Route /upload-img terdaftar pada router aplikasi dan konstanta ROUTES
5. Verifikasi typecheck dan build berhasil tanpa error TypeScript maupun bundling
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/fe-021-20260819T151109Z-662d6817.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-19] Task dibuat melalui orchestrator task intake oleh `user`.

---

## Orchestrator Run Log
- [2026-08-19T15:11:09.217Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T15:11:09.437Z] Run `fe-021-20260819T151109Z-662d6817` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T15:14:13.632Z] Run `fe-021-20260819T151109Z-662d6817`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:fe-021-20260819T151109Z-662d6817 -->
- Classification: `PROJECT_ONLY`
- Summary: Implementasi dan UI slicing halaman Upload Image (/upload-img) pada starter-app dengan integrasi router, layout header/hero/dropzone/footer berbasis aset lokal, serta perbaikan import type-only pada verbatimModuleSyntax.
- Rationale: Task FE-021 adalah implementasi fitur slicing UI spesifik halaman /upload-img (header, hero, dropzone, footer) dan registrasi route di starter-app berdasarkan mockup visual lokal. Tidak ada pola arsitektur baru atau konsep global yang perlu dipromosikan ke LLM Wiki 01-Knowledge/.
- Source: [[03-Sources/other/orchestrator-runs/fe-021-20260819T151109Z-662d6817.json]]
- [2026-08-19T15:16:26.371Z] Run `fe-021-20260819T151109Z-662d6817`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
