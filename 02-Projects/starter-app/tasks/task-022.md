---
title: "Hapus Background Image pada Card Login Form"
type: task
task_id: FE-022
project: starter-app
status: DONE
tags: [task, starter-app, orchestrator-intake]
created: 2026-08-20
updated: 2026-08-20
dependencies: []
verification: ["typecheck", "build"]
allowed_paths: ["src/pages/Login/components/LoginForm.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Hapus Background Image pada Card Login Form

## Permintaan User

di halaman login hilangkan background image di dalam card login form

## Tujuan

Menghilangkan background image pada card/kontainer login form di halaman Login sesuai permintaan user.

## Scope

- `src/pages/Login/components/LoginForm.tsx`

## Hasil Yang Diharapkan

Background image di dalam card login form berhasil dihilangkan sehingga card tidak lagi menampilkan gambar latar `bg-login.png`.

## Acceptance Criteria

1. Background image (`bg-login.png`) pada kontainer kartu di `LoginForm.tsx` dihapus
2. Import `bgLoginImg` yang tidak terpakai dibersihkan dari `LoginForm.tsx`
3. Tampilan kartu login form tetap rapi dengan styling layout dan shadow yang sesuai
4. Verifikasi typecheck dan build berjalan sukses tanpa error TypeScript/build
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/fe-022-20260820T031853Z-9d33055e.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-20] Task dibuat melalui orchestrator task intake oleh `user`.

---

## Orchestrator Run Log
- [2026-08-20T03:18:53.461Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-20T03:18:53.627Z] Run `fe-022-20260820T031853Z-9d33055e` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-20T03:19:34.457Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T03:33:53.629Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN UI FEEDBACK) ===
[Pin #1 pada koordinat X:21% Y:33%]: ganti background color card menjadi warna putih
- [2026-08-20T03:34:14.197Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T03:38:57.880Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN UI FEEDBACK) ===
[Pin #1 pada koordinat X:33% Y:75%]: hapus logo ini
- [2026-08-20T03:40:23.345Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T03:47:33.786Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN & AREA SELECTION) ===
[Area Box #1 posisi X:32% Y:72%, Ukuran 5%×6%]: hapus logo ini
- [2026-08-20T03:48:06.581Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T03:58:27.454Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN & AREA SELECTION) ===
[Area Box #1 posisi X:5% Y:7%, Ukuran 40%×86%]: ganti bg color ke warna abu-abu
- [2026-08-20T03:59:32.498Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T04:04:14.434Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN & AREA SELECTION) ===
[Area Box #1 posisi X:5% Y:7%, Ukuran 40%×86%]: ganti bg color card menjadi dark gray
- [2026-08-20T04:04:43.643Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-20T04:05:12.575Z] Human `user` meminta revisi run `fe-022-20260820T031853Z-9d33055e`: Perbaiki implementasi sesuai dengan anotasi visual & catatan diff terlampir.

=== CATATAN VISUAL REVIEW (PIN & AREA SELECTION) ===
[Area Box #1 posisi X:6% Y:8%, Ukuran 39%×85%]: ganti warna bg color menjadi putih dan buat ada shadownya di card itu
- [2026-08-20T04:05:49.577Z] Run `fe-022-20260820T031853Z-9d33055e`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:fe-022-20260820T031853Z-9d33055e -->
- Classification: `PROJECT_ONLY`
- Summary: FE-022 melakukan modifikasi murni internal komponen/halaman/konfigurasi project (starter-app). Diklasifikasikan secara deterministik sebagai PROJECT_ONLY.
- Rationale: Perubahan cakupan file berada di dalam lapisan presentasi/konfigurasi/pengujian internal project tanpa abstraksi generic yang reusable untuk global knowledge vault.
- Source: [[03-Sources/other/orchestrator-runs/fe-022-20260820T031853Z-9d33055e.json]]
- [2026-08-20T04:13:02.177Z] Run `fe-022-20260820T031853Z-9d33055e`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
