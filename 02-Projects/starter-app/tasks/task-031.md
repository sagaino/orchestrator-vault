---
title: "Integrate UserProfileBadge into Dashboard Page"
type: task
task_id: FE-031
project: starter-app
status: BACKLOG
tags: [task, starter-app, orchestrator-intake]
created: 2026-09-02
updated: 2026-09-02
dependencies: ["FE-030"]
verification: ["typecheck", "build"]
allowed_paths: ["src/pages/Dashboard/index.tsx"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260902043825-01e4faef"
plan_id: "plan-obj-20260902043825-01e4faef-r1"
orchestration_id: "orch-obj-20260902043825-01e4faef"
node_id: "task-integrate-user-profile-badge-dashboard"
master_task: "obj-20260902043825-01e4faef"
orchestration_managed: true
orchestration_dependencies: ["starter-app:FE-030"]
role: "FRONTEND"
node_type: "IMPLEMENTATION"
write_conflict_group: "pages-dashboard"
context_from: ["task-create-user-profile-badge"]
skill_assignments: ["frontend-vite@1.0.0#bde6d8aa3f18070d43d2124d250edec6622d08e3cb03e4a17eb60979c39bce21"]
---

# Integrate UserProfileBadge into Dashboard Page

## Permintaan User

Tolong buatkan komponen baru UserProfileBadge di file src/components/UserProfileBadge.tsx yang menampilkan avatar user, nama akun "Admin AI OS", dan status badge "Online" menggunakan komponen Shadcn UI. Pasang komponen ini di bagian atas halaman src/pages/Dashboard/index.tsx.

Orchestration node: task-integrate-user-profile-badge-dashboard

## Tujuan

Mount UserProfileBadge at the top of src/pages/Dashboard/index.tsx.

## Scope

- `src/pages/Dashboard/index.tsx`

## Hasil Yang Diharapkan

Node task-integrate-user-profile-badge-dashboard memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260902043825-01e4faef-r1.

## Acceptance Criteria

1. Import and place UserProfileBadge at the top section of src/pages/Dashboard/index.tsx
2. Maintain clean layout and strict TypeScript compliance with zero errors
3. Pass typecheck and build verification
4. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
5. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

Belum ditentukan oleh retrospective orchestrator.

## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan

🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]
- [2026-09-02] Task dibuat melalui orchestrator task intake oleh `orchestrator`.
