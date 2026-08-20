---
title: "Tambahkan Tab Harvest from Codebase pada Knowledge Ingest Studio"
type: task
task_id: TASK-018
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/services/orchestrator.ts", "src/hooks/use-orchestrator.ts", "src/pages/Knowledge/components/KnowledgeIngestModal.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Tambahkan Tab Harvest from Codebase pada Knowledge Ingest Studio

## Permintaan User

Tambahkan tab "Harvest from Codebase" pada Knowledge Ingest Studio di orchestrator-dashboard.

Persyaratan:
1. Di `src/services/orchestrator.ts`:
   - Tambahkan fungsi API `harvestKnowledge(payload: { repositoryPath: string; domain?: string })` -> POST `/api/knowledge/harvest`.
2. Di `src/hooks/use-orchestrator.ts`:
   - Tambahkan mutation hook `useHarvestKnowledge()`.
3. Di modal `KnowledgeIngestModal.tsx`:
   - Tambahkan Tab 2: "Harvest from Codebase / Repo".
   - Input: Path Repositori Lokal (misal: `/Users/.../my-backend-service`) & Selector Domain (Backend, Mobile, DevOps, Frontend).
   - Tampilkan progress spinner saat AI menganalisis arsitektur dan feedback daftar artikel knowledge baru yang berhasil dibuat.
4. Pastikan `npm run typecheck` dan `npm run build` lolos 100%.

## Tujuan

Menambahkan fungsionalitas harvesting knowledge otomatis dari repositori lokal ke dalam Knowledge Ingest Studio pada orchestrator-dashboard.

## Scope

- `src/services/orchestrator.ts`
- `src/hooks/use-orchestrator.ts`
- `src/pages/Knowledge/components/KnowledgeIngestModal.tsx`

## Hasil Yang Diharapkan

Tab Harvest from Codebase terintegrasi di modal Knowledge Ingest Studio yang memanggil endpoint API /api/knowledge/harvest via hook useHarvestKnowledge, lengkap dengan input form, visual loading spinner saat analisis repositori, dan feedback daftar artikel hasil harvest.

## Acceptance Criteria

1. src/services/orchestrator.ts mengekspor payload/response type dan fungsi harvestKnowledge() yang memanggil POST /api/knowledge/harvest
2. src/hooks/use-orchestrator.ts mengekspor mutation hook useHarvestKnowledge() menggunakan TanStack Query
3. src/pages/Knowledge/components/KnowledgeIngestModal.tsx menyediakan Tab 2: Harvest from Codebase / Repo dengan form input path repositori dan selector domain (Backend, Mobile, DevOps, Frontend, dll.)
4. Menampilkan state loading / spinner saat proses harvesting berjalan dan menampilkan feedback daftar artikel knowledge baru yang berhasil dibuat
5. Verifikasi typecheck, lint, dan build lolos 100% tanpa error
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `UPDATE`
- Destination: `WIKI`
- Source: Task 018 20260819T020614Z Ab205748.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-19T02:06:10.118Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-19T02:06:14.557Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T02:06:14.614Z] Run `task-018-20260819T020614Z-ab205748` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T02:08:31.361Z] Run `task-018-20260819T020614Z-ab205748`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T02:10:01.902Z] Run `task-018-20260819T020614Z-ab205748`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
