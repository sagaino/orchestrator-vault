---
title: "Tambahkan HTTP API endpoints untuk Project Onboarding di src/server.mjs"
type: task
task_id: TASK-005
project: personal-ai-orchestrator
status: DONE
tags: [task, personal-ai-orchestrator, orchestrator-intake]
created: 2026-08-17
updated: 2026-08-17
dependencies: []
verification: ["test"]
allowed_paths: ["src/project-onboarding.mjs", "src/server.mjs", "test/api.test.mjs"]
requires_changes: true
risk: LOW
sources: []
---

# Tambahkan HTTP API endpoints untuk Project Onboarding di src/server.mjs

## Permintaan User

Tambahkan HTTP API endpoints untuk Project Onboarding di src/server.mjs agar dashboard web bisa mendaftarkan project baru atau meng-onboard project existing.

Persyaratan:
1. Di src/server.mjs, import helper onboarding dari src/project-onboarding.mjs:
   - onboardExistingProject
   - onboardNewProject
2. Buat endpoint POST /api/projects/onboard/existing:
   - Body request: { repositoryPath, projectId? }
   - Validasi keberadaan repositoryPath dan panggil onboardExistingProject({ vaultRoot, runsRoot, repositoryPath, projectId, ... })
   - Broadcast event SSE: "PROJECT_ONBOARDED" dengan payload { projectId, mode: "existing" }
   - Kembalikan HTTP 201 dengan JSON hasil registrasi.
3. Buat endpoint POST /api/projects/onboard/new:
   - Body request: { projectId, targetDirectory, blueprint = "frontend-vite" }
   - Validasi parameter dan panggil onboardNewProject({ vaultRoot, runsRoot, projectId, targetDirectory, blueprint, ... })
   - Broadcast event SSE: "PROJECT_ONBOARDED" dengan payload { projectId, mode: "new" }
   - Kembalikan HTTP 201 dengan JSON hasil pembuatan project.
4. Tangani error dengan try/catch dan sendError(res, statusCode, err.message) jika validasi atau eksekusi onboarding gagal.
5. Pastikan semua unit test di test/api.test.mjs tetap lolos dan tambahkan test coverage untuk kedua endpoint baru tersebut.

## Tujuan

Menambahkan HTTP API endpoints untuk Project Onboarding (existing & new) di server API orchestrator beserta SSE event broadcast dan unit test coverage.

## Scope

- `src/project-onboarding.mjs`
- `src/server.mjs`
- `test/api.test.mjs`

## Hasil Yang Diharapkan

REST API orchestrator menyediakan endpoint POST /api/projects/onboard/existing dan POST /api/projects/onboard/new dengan validasi input, integrasi service onboarding, SSE broadcast PROJECT_ONBOARDED, respons HTTP 201, serta test coverage lengkap di test/api.test.mjs.

## Acceptance Criteria

1. src/project-onboarding.mjs mengekspor onboardExistingProject dan onboardNewProject (sebagai fungsi/alias addExistingProject dan addNewProject)
2. src/server.mjs mengimplementasikan endpoint POST /api/projects/onboard/existing dengan validasi repositoryPath, eksekusi onboardExistingProject, broadcast SSE event PROJECT_ONBOARDED ({ projectId, mode: "existing" }), dan mengembalikan HTTP 201
3. src/server.mjs mengimplementasikan endpoint POST /api/projects/onboard/new dengan validasi projectId & targetDirectory (default blueprint: "frontend-vite"), eksekusi onboardNewProject, broadcast SSE event PROJECT_ONBOARDED ({ projectId, mode: "new" }), dan mengembalikan HTTP 201
4. Error handling menangani input tidak valid (HTTP 400) dan kegagalan proses onboarding dengan sendError(res, statusCode, err.message)
5. test/api.test.mjs menambahkan test coverage untuk kedua endpoint onboarding dan seluruh test suite npm run test tetap lolos (PASS)
6. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
7. Verification `test` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: Task 005 20260817T125127Z 57aa5dc3.json


## Error Log

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

---

## Orchestrator Intake Log
- [2026-08-17T12:51:26.719Z] Task dibuat dari conversational intake oleh `user`; risk `LOW`.

---

## Orchestrator Run Log
- [2026-08-17T12:51:27.897Z] Human `user` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-08-17T12:51:27.956Z] Run `task-005-20260817T125127Z-57aa5dc3` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-08-17T12:53:58.202Z] Run `task-005-20260817T125127Z-57aa5dc3`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:task-005-20260817T125127Z-57aa5dc3 -->
- Classification: `PROJECT_ONLY`
- Summary: Menambahkan HTTP API endpoints POST /api/projects/onboard/existing dan POST /api/projects/onboard/new di src/server.mjs dengan integrasi helper src/project-onboarding.mjs dan event broadcast SSE PROJECT_ONBOARDED.
- Rationale: Task TASK-005 menambahkan endpoint POST /api/projects/onboard/existing dan POST /api/projects/onboard/new ke dalam server.mjs orchestrator lokal, beserta unit test di test/api.test.mjs. Logic onboarding itu sendiri sudah didefinisikan dalam blueprint & arsitektur AGENTS.md, sehingga penambahan rute API ini merupakan implementasi spesifik proyek personal-ai-orchestrator.
- Source: Task 005 20260817T125127Z 57aa5dc3.json
- [2026-08-17T12:54:23.409Z] Run `task-005-20260817T125127Z-57aa5dc3`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
