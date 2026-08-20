---
title: "Implementasi UI Add / Onboard Project Modal di Dashboard"
type: task
task_id: TASK-014
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-17
updated: 2026-08-17
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/services/orchestrator.ts", "src/hooks/use-orchestrator.ts", "src/components/project/AddProjectModal.tsx", "src/components/layout/DashboardLayout.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Implementasi UI Add / Onboard Project Modal di Dashboard

## Permintaan User

Buat fitur UI "Add / Onboard Project" di orchestrator-dashboard dengan modal dialog interaktif.

Persyaratan:
1. Di src/services/orchestrator.ts:
   - Tambahkan fungsi API onboardExistingProject(payload: { repositoryPath: string; projectId?: string }) -> POST /api/projects/onboard/existing
   - Tambahkan fungsi API onboardNewProject(payload: { projectId: string; targetDirectory: string; blueprint?: string }) -> POST /api/projects/onboard/new
2. Di src/hooks/use-orchestrator.ts:
   - Buat mutation hook useOnboardExistingProject() dan useOnboardNewProject() yang otomatis menginvalidasi cache queryKeys.projects saat sukses.
3. Buat komponen modal src/components/project/AddProjectModal.tsx:
   - Tab 1: "Existing Project" (Input: Repository Path lokal & Optional Project ID)
   - Tab 2: "New Project (Vite + Shadcn)" (Input: Project ID & Target Directory)
   - Tampilkan loading spinner / progress state saat proses onboarding berlangsung.
   - Tampilkan error alert jika backend mengembalikan pesan error.
4. Pasang tombol trigger "+ Add Project" di sidebar navigasi atau top header DashboardLayout.tsx.
5. Pastikan npm run typecheck dan npm run build lolos 100%.

## Tujuan

Menyediakan antarmuka modal interaktif untuk onboarding project baru maupun existing ke orchestrator backend langsung dari dashboard UI.

## Scope

- `src/services/orchestrator.ts`
- `src/hooks/use-orchestrator.ts`
- `src/components/project/AddProjectModal.tsx`
- `src/components/layout/DashboardLayout.tsx`

## Hasil Yang Diharapkan

Pengguna dapat membuka modal 'Add Project' dari DashboardLayout dan melakukan onboarding project (existing path atau project baru) dengan loading feedback, error handling, dan auto-refresh daftar project.

## Acceptance Criteria

1. Fungsi onboardExistingProject dan onboardNewProject diimplementasikan pada src/services/orchestrator.ts dengan endpoint POST /api/projects/onboard/existing dan POST /api/projects/onboard/new.
2. Mutation hooks useOnboardExistingProject dan useOnboardNewProject ditambahkan pada src/hooks/use-orchestrator.ts serta menginvalidasi queryKeys.projects saat mutasi sukses.
3. Komponen src/components/project/AddProjectModal.tsx dibuat dengan 2 tab (Existing Project dan New Project), form input yang sesuai, visual loading state, dan error alert.
4. Tombol trigger '+ Add Project' terpasang pada src/components/layout/DashboardLayout.tsx untuk membuka AddProjectModal.
5. Verifikasi typecheck, lint, dan build berhasil tanpa error.
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: Task 014 20260817T125505Z Ab395e50.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-17T12:55:00.955Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-17T12:55:05.708Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-17T12:55:05.766Z] Run `task-014-20260817T125505Z-ab395e50` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-17T12:58:25.614Z] Run `task-014-20260817T125505Z-ab395e50`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-014-20260817T125505Z-ab395e50 -->
- Classification: `PROJECT_ONLY`
- Summary: Implementasi AddProjectModal dengan 2 tab onboarding (existing repo path & new Vite template), hook useOnboardExistingProject / useOnboardNewProject, integrasi trigger di DashboardLayout, serta pembersihan ESLint purity rule.
- Rationale: Fitur onboarding modal dan mutation hooks yang dibuat khusus untuk kebutuhan navigasi dan manajemen repository proyek pada dashboard Personal AI Orchestrator. Perbaikan purity lint adalah best practice standar React yang telah ditangani dan tidak memerlukan modifikasi knowledge global.
- Source: Task 014 20260817T125505Z Ab395e50.json
- [2026-08-17T12:58:55.990Z] Run `task-014-20260817T125505Z-ab395e50`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
