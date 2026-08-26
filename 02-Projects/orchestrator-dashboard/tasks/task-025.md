---
title: "Hapus Karakter Plus Duplikat pada Tombol Add Project Sidebar"
type: task
task_id: TASK-025
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-21
updated: 2026-08-21
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/components/layout/DashboardLayout.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Hapus Karakter Plus Duplikat pada Tombol Add Project Sidebar

## Permintaan User

coba cek button add project, ada dua icon + hapus 1 yang paling kecil

## Tujuan

Menghilangkan karakter '+' duplikat yang berukuran lebih kecil pada label tombol Add Project di sidebar sehingga hanya menyisakan icon Lucide Plus dan teks 'Add Project'.

## Scope

- `src/components/layout/DashboardLayout.tsx`

## Hasil Yang Diharapkan

Tombol Add Project di sidebar DashboardLayout hanya menampilkan satu ikon '+' (icon Lucide Plus) dan label teks 'Add Project' tanpa karakter '+' literal tambahan di dalam tag span.

## Acceptance Criteria

1. Teks pada tombol Add Project di sidebar DashboardLayout tidak lagi memiliki karakter '+' duplikat di dalam label teks (menjadi 'Add Project')
2. Icon Lucide Plus tetap tampil dengan benar di samping label teks tanpa duplikasi icon/simbol tambah
3. Proyek lolos verifikasi lint, typecheck, dan build tanpa error
4. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
5. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/task-025-20260821T043059Z-a321cd66.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-08-21] Task dibuat melalui orchestrator task intake oleh `user`.

---

## Orchestrator Run Log
- [2026-08-21T04:30:59.422Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-21T04:30:59.557Z] Run `task-025-20260821T043059Z-a321cd66` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-21T04:31:36.285Z] Run `task-025-20260821T043059Z-a321cd66`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-025-20260821T043059Z-a321cd66 -->
- Classification: `PROJECT_ONLY`
- Summary: TASK-025 melakukan modifikasi murni internal komponen/halaman/konfigurasi project (orchestrator-dashboard). Diklasifikasikan secara deterministik sebagai PROJECT_ONLY.
- Rationale: Perubahan cakupan file berada di dalam lapisan presentasi/konfigurasi/pengujian internal project tanpa abstraksi generic yang reusable untuk global knowledge vault.
- Source: [[03-Sources/other/orchestrator-runs/task-025-20260821T043059Z-a321cd66.json]]
- [2026-08-21T04:37:19.720Z] Run `task-025-20260821T043059Z-a321cd66`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
