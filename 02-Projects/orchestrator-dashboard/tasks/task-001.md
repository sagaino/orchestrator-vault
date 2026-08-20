---
title: "Refactor route path tasks menggunakan konstanta ROUTES.TASK"
type: task
task_id: TASK-001
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-16
updated: 2026-08-16
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/lib/constant/routes.ts", "src/routes/index.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Refactor route path tasks menggunakan konstanta ROUTES.TASK

## Permintaan User

gunakan constant routes di routes/index.tsx karena sekarang path masih seperti ini : 
path: "tasks". saya maunya nanti path: ROUTES.TASK

## Tujuan

Mengganti hardcoded string path "tasks" pada konfigurasi route di `src/routes/index.tsx` dengan konstanta `ROUTES.TASK` yang terpusat di `src/lib/constant/routes.ts`.

## Scope

- `src/lib/constant/routes.ts`
- `src/routes/index.tsx`

## Hasil Yang Diharapkan

Route task di `src/routes/index.tsx` dikonfigurasi menggunakan konstanta `ROUTES.TASK` yang didefinisikan di `src/lib/constant/routes.ts`.

## Acceptance Criteria

1. Konstanta `TASK` ditambahkan pada objek `ROUTES` di `src/lib/constant/routes.ts`
2. `src/routes/index.tsx` mengimpor `ROUTES` dari `@/lib/constant/routes` (atau path relatifnya)
3. Properti path untuk route task di `src/routes/index.tsx` menggunakan `ROUTES.TASK` menggantikan string literal "tasks"
4. Lolos verifikasi `typecheck`, `lint`, dan `build` tanpa error
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: Task 001 20260816T050831Z 67e88998.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-16T05:03:35.936Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-16T05:03:36.741Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-16T05:03:36.785Z] Run `task-001-20260816T050336Z-319478cd` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-16T05:08:30.936Z] Human `user` meminta retry setelah run `task-001-20260816T050336Z-319478cd`: force retry setelah human review (Unknown failure).
- [2026-08-16T05:08:31.060Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-16T05:09:32.293Z] Run `task-001-20260816T050831Z-67e88998` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-16T05:10:36.262Z] Run `task-001-20260816T050831Z-67e88998`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-001-20260816T050831Z-67e88998 -->
- Classification: `PROJECT_ONLY`
- Summary: Refaktorisasi route path 'tasks' menjadi konstanta terpusat ROUTES.TASK pada project orchestrator-dashboard.
- Rationale: Perubahan ini merupakan refaktorisasi rute internal spesifik pada orchestrator-dashboard untuk memusatkan rute task ke konstanta ROUTES.TASK di src/lib/constant/routes.ts dan menerapkannya pada src/routes/index.tsx. Perubahan ini mematuhi konvensi yang sudah ada tanpa memperkenalkan pola arsitektur atau snippet baru yang dapat digeneralisasi ke wiki global.
- Source: Task 001 20260816T050831Z 67e88998.json
- [2026-08-16T05:14:17.684Z] Run `task-001-20260816T050831Z-67e88998`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
