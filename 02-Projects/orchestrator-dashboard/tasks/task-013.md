---
title: "Integrasi Fitur Auto-Fix Safe Vault Health pada Knowledge Center"
type: task
task_id: TASK-013
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-17
updated: 2026-08-17
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/services/orchestrator.ts", "src/hooks/use-orchestrator.ts", "src/pages/Knowledge/components/VaultHealthCard.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Integrasi Fitur Auto-Fix Safe Vault Health pada Knowledge Center

## Permintaan User

Di project orchestrator-dashboard, integrasikan fitur Auto-Fix Safe pada halaman Knowledge Center:
1. Pada src/services/orchestrator.ts dan src/hooks/use-orchestrator.ts, tambahkan method dan mutation hook useFixSafeKnowledgeHealth() yang memanggil endpoint POST /api/knowledge/health/fix-safe.
2. Pada komponen VaultHealthCard (src/pages/Knowledge/components/VaultHealthCard.tsx), tambahkan tombol 'Auto-Fix Safe Issues' dengan icon Wrench/Sparkles yang muncul saat terdapat unindexed files atau safe warnings.
3. Tampilkan toast konfirmasi keberhasilan setelah proses perbaikan selesai dan otomatis refresh data useKnowledgeHealth.
4. Pastikan lolos verifikasi (npm run typecheck, npm run lint, npm run build).

## Tujuan

Mengintegrasikan fitur Auto-Fix Safe pada halaman Knowledge Center dashboard untuk memungkinkan perbaikan otomatis isu-isu aman pada Vault Knowledge seperti pendaftaran file unindexed ke index.md secara instan melalui UI.

## Scope

- `src/services/orchestrator.ts`
- `src/hooks/use-orchestrator.ts`
- `src/pages/Knowledge/components/VaultHealthCard.tsx`

## Hasil Yang Diharapkan

Fitur Auto-Fix Safe terintegrasi penuh di halaman Knowledge Center dengan endpoint API, mutation hook, tombol aksi reaktif pada VaultHealthCard saat ada unindexed files/warnings, toast notifikasi hasil eksekusi, dan auto-refetch health data.

## Acceptance Criteria

1. Method fixSafeKnowledgeHealth() ditambahkan pada OrchestratorApi di src/services/orchestrator.ts untuk memanggil endpoint POST /api/knowledge/health/fix-safe.
2. Hook useFixSafeKnowledgeHealth() ditambahkan pada src/hooks/use-orchestrator.ts dan meng-invalidate query knowledgeHealth setelah mutasi sukses.
3. Tombol 'Auto-Fix Safe Issues' (dengan icon Wrench atau Sparkles) ditampilkan di VaultHealthCard saat unindexedCount > 0 atau terdapat safe warnings.
4. Toast konfirmasi sukses/gagal ditampilkan saat proses Auto-Fix Safe selesai dan data Vault Health otomatis ter-refresh.
5. Lolos verifikasi typecheck, lint, dan build tanpa error.
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: Task 013 20260817T025730Z 4c8c7a83.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-17T02:57:28.851Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-17T02:57:30.572Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-17T02:57:30.622Z] Run `task-013-20260817T025730Z-4c8c7a83` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-17T02:59:19.461Z] Run `task-013-20260817T025730Z-4c8c7a83`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-013-20260817T025730Z-4c8c7a83 -->
- Classification: `PROJECT_ONLY`
- Summary: Integrasi fitur Auto-Fix Safe Vault Health pada Orchestrator Dashboard melalui API endpoint POST /api/knowledge/health/fix-safe, custom mutation hook TanStack Query useFixSafeKnowledgeHealth, dan tombol interaktif Auto-Fix Safe pada VaultHealthCard dengan notifikasi toast dan auto-refetch health data.
- Rationale: Perubahan pada TASK-013 adalah integrasi fitur Auto-Fix Safe spesifik pada Orchestrator Dashboard (menambahkan API call POST /api/knowledge/health/fix-safe, hook useFixSafeKnowledgeHealth dengan query invalidation, dan tombol 'Auto-Fix Safe Issues' di VaultHealthCard). Pola arsitektur API -> Hook -> UI yang diterapkan sudah sepenuhnya terdokumentasi di global knowledge base (API Service Layer & Data Flow). Oleh karena itu, insight dari task ini bersifat PROJECT_ONLY dan tidak memerlukan pembuatan atau modifikasi file knowledge global.
- Source: Task 013 20260817T025730Z 4c8c7a83.json
- [2026-08-17T03:00:05.731Z] Run `task-013-20260817T025730Z-4c8c7a83`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
