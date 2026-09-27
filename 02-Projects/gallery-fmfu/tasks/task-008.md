---
title: "Conditionally Set Media Endpoints in endpoint.ts"
type: task
task_id: GFM-008
project: gallery-fmfu
status: DONE
tags: [task, gallery-fmfu, orchestrator-intake]
created: 2026-09-01
updated: 2026-09-01
dependencies: []
verification: ["typecheck", "build"]
allowed_paths: ["src/lib/constant/endpoint.ts"]
requires_changes: true
risk: LOW
complexity: LOW
sources: []
objective_id: "obj-20260901040718-46692eec"
plan_id: "plan-obj-20260901040718-46692eec-r1"
orchestration_id: "orch-obj-20260901040718-46692eec"
node_id: "node-1"
master_task: "obj-20260901040718-46692eec"
orchestration_managed: true
orchestration_dependencies: []
role: "FRONTEND"
node_type: "IMPLEMENTATION"
write_conflict_group: null
context_from: []
skill_assignments: ["frontend-vite@1.0.0#bde6d8aa3f18070d43d2124d250edec6622d08e3cb03e4a17eb60979c39bce21"]
---

# Conditionally Set Media Endpoints in endpoint.ts

## Permintaan User

buat condition dimana jika name adalah Alice Putri (dibuat lower case dan di jadikan array) maka di src/lib/constant/endpoint.ts untuk POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL,
POST_MEDIA_DOWNLOAD menggunakan media-mapping. jika bukan maka menggunakan post-media. untuk name bisa di cek di localstorage dengan key user

Orchestration node: node-1

## Tujuan

Update POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, and POST_MEDIA_DOWNLOAD in src/lib/constant/endpoint.ts to switch between media-mapping and post-media based on user name in localStorage.

## Scope

- `src/lib/constant/endpoint.ts`

## Hasil Yang Diharapkan

Node node-1 memenuhi seluruh acceptance criteria yang disetujui pada plan plan-obj-20260901040718-46692eec-r1.

## Acceptance Criteria

1. Retrieve user name from localStorage key 'user' and convert to lowercase
2. Check inclusion in an array containing 'alice putri'
3. Use 'media-mapping' endpoint for POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, and POST_MEDIA_DOWNLOAD if matched, otherwise 'post-media'
4. Pass typecheck and build verifications
5. Hanya file dalam `allowed_paths` yang berubah akibat task ini.
6. Verification `typecheck` dan `build` berhasil.

## Knowledge Decision

- Classification: `PROJECT_ONLY`
- Destination: `PROJECT`
- Source: [[03-Sources/other/orchestrator-runs/gfm-008-20260901T040805Z-df76042c.json]]


## Error Log

Tidak ada error log saat pembuatan task.

## Log Perubahan
🚀 [VERIFIED_BY_LLM_WIKI_SCHEMA]

- [2026-09-01] Task dibuat melalui orchestrator task intake oleh `orchestrator`.

---

## Orchestrator Run Log
- [2026-09-01T04:08:04.957Z] Human `local:sagaino` memberi approval execution melalui `start-task`: `BACKLOG → READY`.
- [2026-09-01T04:08:05.198Z] Run `gfm-008-20260901T040805Z-df76042c` melakukan atomic claim: `READY → IN_PROGRESS`.
- [2026-09-01T04:09:18.139Z] Run `gfm-008-20260901T040805Z-df76042c`: coding agent, verification, dan Graphify selesai; menunggu human review.
- [2026-09-01T04:11:05.766Z] Human `local:sagaino` meminta revisi run `gfm-008-20260901T040805Z-df76042c`: untuk isi dari ALLOWED_USERS tetap Alice Putri jangan di buat huruf kecil semua tapi ketika ALLOWED_USERS di gunakan baru di lower case
- [2026-09-01T04:11:27.837Z] Run `gfm-008-20260901T040805Z-df76042c`: coding agent, verification, dan Graphify selesai; menunggu human review.

## Knowledge Retrospective

<!-- orchestrator-run:gfm-008-20260901T040805Z-df76042c -->
- Classification: `PROJECT_ONLY`
- Summary: Menambahkan dynamic endpoint switching pada src/lib/constant/endpoint.ts gallery-fmfu untuk POST_MEDIA_ALL, POST_MEDIA_THUMBNAIL, dan POST_MEDIA_DOWNLOAD berdasarkan username pengguna di localStorage.
- Rationale: Implementasi switching endpoint 'media-mapping' vs 'post-media' berdasarkan username 'Alice Putri' di localStorage merupakan logika bisnis spesifik untuk project gallery-fmfu dan tidak memenuhi kriteria promosi global knowledge layer (01-Knowledge/).
- Source: [[03-Sources/other/orchestrator-runs/gfm-008-20260901T040805Z-df76042c.json]]
- [2026-09-01T04:12:45.141Z] Run `gfm-008-20260901T040805Z-df76042c`: human approval, verification, dan knowledge decision lengkap; task ditutup sebagai DONE.
