---
title: "Update LoginForm Card Background and Add Partner Logos"
type: task
task_id: FE-020
project: starter-app
status: DONE
tags: [task, starter-app, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["typecheck", "build"]
allowed_paths: ["src/pages/Login/components/LoginForm.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Update LoginForm Card Background and Add Partner Logos

## Permintaan User

di halaman login src/pages/Login. untuk bagian LoginForm kan ada cardnya, ganti bg card dengan bg-login.png dan di bawah button search tambahkan icon logo dari chatgpt-image-jan-13--2026--11_08_13 dan identifikasi-logo.png di tengah dengan gap 20px

## Project Production Assets

- File: `src/assets/images/chatgpt-image-jan-13--2026--11_08_13-am.png` (Import: `@/assets/images/chatgpt-image-jan-13--2026--11_08_13-am.png`)
- File: `src/assets/images/identifikasi-logo.png` (Import: `@/assets/images/identifikasi-logo.png`)
- File: `src/assets/images/bg-login.png` (Import: `@/assets/images/bg-login.png`)

## Tujuan

Mengganti background card LoginForm dengan asset bg-login.png serta menambahkan logo chatgpt-image-jan-13--2026--11_08_13-am.png dan identifikasi-logo.png di bawah tombol SEARCH dengan posisi tengah dan gap 20px.

## Scope

- `src/pages/Login/components/LoginForm.tsx`

## Hasil Yang Diharapkan

Latar belakang card LoginForm menampilkan gambar bg-login.png dan di bawah tombol submit SEARCH terdapat dua logo (chatgpt-image-jan-13--2026--11_08_13-am.png dan identifikasi-logo.png) yang tersusun rapi di posisi tengah dengan gap 20px.

## Acceptance Criteria

1. Card LoginForm menggunakan background image dari asset @/assets/images/bg-login.png (cover/center) menggantikan background statis sebelumnya
2. Di bawah tombol submit SEARCH terdapat kontainer berisi dua logo: @/assets/images/chatgpt-image-jan-13--2026--11_08_13-am.png dan @/assets/images/identifikasi-logo.png
3. Kedua logo ditampilkan secara horizontal di tengah (centered) dengan jarak (gap) sebesar 20px
4. Proyek berhasil lulus verifikasi typecheck dan build tanpa error
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/fe-020-20260819T150350Z-c23b9cc5.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-19] Task dibuat melalui orchestrator task intake oleh `user`.

---

## Orchestrator Run Log
- [2026-08-19T15:03:50.098Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T15:03:50.267Z] Run `fe-020-20260819T150350Z-c23b9cc5` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T15:05:14.420Z] Run `fe-020-20260819T150350Z-c23b9cc5`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:fe-020-20260819T150350Z-c23b9cc5 -->
- Classification: `PROJECT_ONLY`
- Summary: FE-020 melakukan modifikasi murni internal komponen/halaman/konfigurasi project (starter-app). Diklasifikasikan secara deterministik sebagai PROJECT_ONLY.
- Rationale: Perubahan cakupan file berada di dalam lapisan presentasi/konfigurasi/pengujian internal project tanpa abstraksi generic yang reusable untuk global knowledge vault.
- Source: [[03-Sources/other/orchestrator-runs/fe-020-20260819T150350Z-c23b9cc5.json]]
- [2026-08-19T15:06:37.761Z] Run `fe-020-20260819T150350Z-c23b9cc5`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
