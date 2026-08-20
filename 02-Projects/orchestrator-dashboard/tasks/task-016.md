---
title: "Implementasi UI Knowledge Ingest Studio di Knowledge Center"
type: task
task_id: TASK-016
project: orchestrator-dashboard
status: DONE
tags: [task, orchestrator-dashboard, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["typecheck", "lint", "build"]
allowed_paths: ["src/services/orchestrator.ts", "src/hooks/use-orchestrator.ts", "src/pages/Knowledge/components/KnowledgeIngestModal.tsx", "src/pages/Knowledge/components/KnowledgeCandidatesList.tsx", "src/pages/Knowledge/components/index.ts", "src/pages/Knowledge/index.tsx"]
requires_changes: true
risk: LOW
sources: []
---

# Implementasi UI Knowledge Ingest Studio di Knowledge Center

## Permintaan User

Buat fitur UI "Knowledge Ingest Studio" di orchestrator-dashboard dengan modal dialog interaktif untuk memasukkan knowledge baru secara cepat.

Persyaratan:
1. Di `src/services/orchestrator.ts`:
   - Tambahkan fungsi API `ingestKnowledge(payload: { content: string; title?: string; domain: string; type: string; destination?: "WIKI" | "CANDIDATE" })` -> POST `/api/knowledge/ingest`.
2. Di `src/hooks/use-orchestrator.ts`:
   - Buat mutation hook `useIngestKnowledge()` yang otomatis menginvalidasi `queryKeys.knowledgeCandidates` dan `queryKeys.knowledgeHealth` saat sukses.
3. Buat komponen modal `src/pages/Knowledge/components/KnowledgeIngestModal.tsx`:
   - Form input:
     - Judul Dokumen (Opsional / AI inferred)
     - Domain Target (Radio/Select: Frontend, Backend, Mobile, DevOps, Architecture, General)
     - Tipe Knowledge (Select: Concept, Pattern, Snippet, Decision, Debugging)
     - Destinasi (Toggle: Langsung ke Wiki vs Simpan sebagai Candidate)
     - Text Area besar untuk paste teks mentah / artikel / code snippet referensi.
   - Status feedback: Tampilkan loading spinner saat AI sedang mensintesis, dan alert sukses saat berhasil.
4. Pasang tombol trigger "+ Ingest Knowledge" di header halaman Knowledge Center (`src/pages/Knowledge/index.tsx`).
5. Tambahkan Filter Tag Domain (All, Frontend, Backend, Mobile, DevOps) pada kartu daftar candidates/knowledge di Knowledge Center.
6. Pastikan `npm run typecheck` dan `npm run build` lolos 100%.

## Tujuan

Menyediakan antarmuka modal dialog 'Knowledge Ingest Studio' pada orchestrator-dashboard agar pengguna dapat memasukkan raw knowledge / catatan secara instan ke Wiki atau Candidates.

## Scope

- `src/services/orchestrator.ts`
- `src/hooks/use-orchestrator.ts`
- `src/pages/Knowledge/components/KnowledgeIngestModal.tsx`
- `src/pages/Knowledge/components/KnowledgeCandidatesList.tsx`
- `src/pages/Knowledge/components/index.ts`
- `src/pages/Knowledge/index.tsx`

## Hasil Yang Diharapkan

Fitur Knowledge Ingest Studio berhasil diintegrasikan ke halaman Knowledge Center, memungkinkan pengguna menginput raw knowledge melalui modal dialog interaktif yang terhubung ke API backend dan otomatis me-refresh cache data knowledge.

## Acceptance Criteria

1. Tambahkan fungsi API `ingestKnowledge` pada `src/services/orchestrator.ts` yang memanggil POST `/api/knowledge/ingest`.
2. Buat custom hook mutation `useIngestKnowledge` di `src/hooks/use-orchestrator.ts` yang menginvalidasi query cache `knowledgeCandidates` dan `knowledgeHealth` ketika sukses.
3. Buat komponen modal `KnowledgeIngestModal.tsx` dengan form lengkap (judul opsional, domain target, tipe knowledge, destinasi WIKI/CANDIDATE, textarea konten mentah) disertai status loading spinner dan alert feedback.
4. Pasang tombol trigger '+ Ingest Knowledge' di header `src/pages/Knowledge/index.tsx` untuk membuka `KnowledgeIngestModal`.
5. Tambahkan filter tag domain (All, Frontend, Backend, Mobile, DevOps, Architecture, General) pada daftar knowledge / candidates.
6. Verifikasi bahwa script `npm run typecheck`, `npm run lint`, dan `npm run build` berjalan sukses tanpa error.
7. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
8. Verification `typecheck` dan `lint` dan `build` berhasil.

## Knowledge Decision

- Classification: `NEW`
- Destination: `WIKI`
- Source: Task 016 20260819T013216Z 8dc91dd0.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-19T01:32:11.362Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-19T01:32:16.278Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T01:32:16.347Z] Run `task-016-20260819T013216Z-8dc91dd0` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T01:34:52.267Z] Run `task-016-20260819T013216Z-8dc91dd0`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T01:36:45.870Z] Human `user` meminta revisi run `task-016-20260819T013216Z-8dc91dd0`: buat modalnya menjadi lebih lebar karena bagian Destinasi Vault masih terlalu mepet
- [2026-08-19T01:37:37.220Z] Run `task-016-20260819T013216Z-8dc91dd0`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T01:40:42.319Z] Human `user` meminta revisi run `task-016-20260819T013216Z-8dc91dd0`: samakan ui untuk non-mobile ke mobile
- [2026-08-19T01:41:44.291Z] Run `task-016-20260819T013216Z-8dc91dd0`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T01:43:32.659Z] Run `task-016-20260819T013216Z-8dc91dd0`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
