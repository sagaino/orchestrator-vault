---
title: "Implementasi Inline Diff Annotations pada Diff Viewer & Request Changes Modal"
type: task
task_id: TASK-017
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/components/diff/DiffViewer.tsx", "src/pages/Runs/types/runs.ts", "src/pages/Runs/components/RequestChangesModal.tsx", "src/pages/Runs/components/RunInspector.tsx", "src/pages/Runs/hooks/useRunsPage.ts", "src/pages/Runs/index.tsx", "src/hooks/use-orchestrator.ts", "src/services/orchestrator.ts"]
requires_changes: true
risk: LOW
sources: []
---

# Implementasi Inline Diff Annotations pada Diff Viewer & Request Changes Modal

## Permintaan User

Buat fitur Inline Diff Annotations pada Diff Viewer di halaman Runs & Review orchestrator-dashboard.

Persyaratan:
1. Di komponen diff viewer `src/pages/Runs/components/DiffViewer.tsx` (atau komponen diff terkait):
   - Tambahkan tombol hover `+` di setiap baris penambahan/perubahan kode untuk menambahkan inline comment.
   - Sediakan form popover / inline input untuk mengetik catatan reviewer pada baris tersebut.
   - Tampilkan badge list inline comments yang aktif di atas/bawah baris kode yang bersangkutan.
2. Di modal / panel "Request Changes":
   - Tampilkan ringkasan seluruh inline comments yang telah ditambahkan reviewer.
   - Kirimkan array `inlineComments: [{ file, line, comment }]` bersama teks feedback saat memanggil `useRequestChanges()`.
3. Pastikan `npm run typecheck` dan `npm run build` lolos 100%.

## Tujuan

Menambahkan fitur Inline Diff Annotations pada Diff Viewer dan integrasi feedback revisi di Request Changes Modal untuk halaman Runs & Review orchestrator-dashboard.

## Scope

- `src/components/diff/DiffViewer.tsx`
- `src/pages/Runs/types/runs.ts`
- `src/pages/Runs/components/RequestChangesModal.tsx`
- `src/pages/Runs/components/RunInspector.tsx`
- `src/pages/Runs/hooks/useRunsPage.ts`
- `src/pages/Runs/index.tsx`
- `src/hooks/use-orchestrator.ts`
- `src/services/orchestrator.ts`

## Hasil Yang Diharapkan

Fitur Inline Diff Annotations terintegrasi penuh di halaman Runs & Review: reviewer dapat memberi catatan inline per baris pada DiffViewer, melihat daftar catatan di Request Changes Modal, dan mengirimkan catatan terstruktur tersebut ke backend agent revision API.

## Acceptance Criteria

1. Tombol hover '+' tersedia pada setiap baris penambahan (+) atau perubahan di DiffViewer untuk memicu popover/input inline comment.
2. Disediakan popover/inline form input untuk menulis dan menyimpan inline comment per file dan baris kode.
3. Badge atau inline comment list aktif ditampilkan secara visual di baris kode yang bersangkutan.
4. Modal Request Changes menampilkan ringkasan daftar seluruh inline comments yang telah dibuat reviewer.
5. Array inlineComments: [{ file, line, comment }] dikirimkan bersama payload teks feedback ke useRequestChanges() dan API requestChanges.
6. Verifikasi typecheck, lint, dan build lolos 100% tanpa error.
7. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
8. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `NEW`
- Destination: `WIKI`
- Source: Task 017 20260819T015532Z 6078715f.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-19T01:55:29.559Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-19T01:55:32.896Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T01:55:32.962Z] Run `task-017-20260819T015532Z-6078715f` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T01:58:05.618Z] Run `task-017-20260819T015532Z-6078715f`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T02:01:11.062Z] Run `task-017-20260819T015532Z-6078715f`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
