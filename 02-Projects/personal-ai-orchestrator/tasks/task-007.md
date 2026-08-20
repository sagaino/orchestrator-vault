---
title: "Dukungan Inline Line Comments pada Siklus Request-Changes dan Run Review"
type: task
task_id: TASK-007
project: personal-ai-orchestrator
status: DONE
tags: [task, personal-ai-orchestrator, orchestrator-intake]
created: 2026-08-19
updated: 2026-08-19
dependencies: []
verification: ["test"]
allowed_paths: ["src/server.mjs", "src/review-workflow.mjs", "src/context-compactor.mjs", "src/executor.mjs", "test/api.test.mjs"]
requires_changes: true
risk: LOW
sources: []
---

# Dukungan Inline Line Comments pada Siklus Request-Changes dan Run Review

## Permintaan User

Tambahkan dukungan inline line comments pada siklus request-changes dan run review di backend personal-ai-orchestrator.

Persyaratan:
1. Di `src/server.mjs`:
   - Pada endpoint `POST /api/runs/:id/request-changes`, terima payload opsional `inlineComments?: Array<{ file: string; line: number; comment: string }>` selain `feedback`.
2. Di `src/review-workflow.mjs`:
   - Teruskan `inlineComments` ke prompt revisi coding agent.
   - Format komentar baris dengan jelas pada prompt:
     "=== INLINE CODE COMMENTS DARI REVIEWER ==="
     File: <file> (Line <line>): "<comment>"
   - Instruksikan agent untuk memprioritaskan perbaikan pada baris-baris spesifik yang diberi catatan oleh reviewer.
3. Pastikan `npm test` lolos 100%.

## Tujuan

Menambahkan dukungan inline line comments pada siklus request-changes dan run review backend personal-ai-orchestrator.

## Scope

- `src/server.mjs`
- `src/review-workflow.mjs`
- `src/context-compactor.mjs`
- `src/executor.mjs`
- `test/api.test.mjs`

## Hasil Yang Diharapkan

Endpoint POST /api/runs/:id/request-changes menerima inlineComments dan review workflow memformat komentar baris secara terstruktur ke dalam prompt revisi coding agent serta lulus verifikasi npm test.

## Acceptance Criteria

1. Endpoint POST /api/runs/:id/request-changes di src/server.mjs menerima inlineComments?: Array<{ file: string, line: number, comment: string }> opsional selain reason/feedback.
2. Komentar baris diteruskan ke prompt revisi coding agent dengan header '=== INLINE CODE COMMENTS DARI REVIEWER ===' dan format 'File: <file> (Line <line>): "<comment>"'.
3. Prompt revisi menginstruksikan coding agent untuk memprioritaskan perbaikan pada baris-baris spesifik yang diberi catatan oleh reviewer.
4. Seluruh pengujian otomatis pada npm test lolos 100%.
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `test` berhasil.

## Knowledge Decision

- Classification: `NEW`
- Destination: `WIKI`
- Source: Task 007 20260819T015030Z 3fb07f02.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-19T01:50:25.457Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-19T01:50:29.997Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-19T01:50:30.054Z] Run `task-007-20260819T015030Z-3fb07f02` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-19T01:53:59.992Z] Run `task-007-20260819T015030Z-3fb07f02`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-08-19T01:54:34.509Z] Run `task-007-20260819T015030Z-3fb07f02`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
